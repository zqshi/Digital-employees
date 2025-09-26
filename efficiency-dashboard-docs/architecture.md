# 效率看板 - 功能架构设计文档

## 📐 整体架构

效率看板采用模块化的组件架构设计，每个功能模块独立开发和维护，通过统一的容器组件进行协调管理。

```
EfficiencyDashboard (主容器)
├── Header (头部控制区)
│   ├── 标题和描述
│   ├── 时间范围选择器
│   ├── 自定义日期范围
│   ├── 刷新控制
│   └── 最后更新时间
└── Content (内容区域)
    ├── ProjectProgressModule (项目进度总览)
    │   ├── 指标卡片组
    │   └── 燃尽图可视化
    ├── StageOutputModule (阶段产出物)
    │   ├── 阶段横向滚动
    │   ├── 文档聚合展示
    │   └── 导航控制
    └── DigitalWorkerStatusModule (数字员工状态)
        ├── 状态统计面板
        ├── 员工列表表格
        └── 筛选排序控制
```

## 🏗️ 组件层次结构

### 主容器层 (Container Layer)
**EfficiencyDashboard.tsx**
- 负责整体状态管理
- 时间范围控制逻辑
- 自动刷新机制
- 子组件数据传递

### 功能模块层 (Module Layer)

#### 1. ProjectProgressModule
```typescript
interface ProjectProgressModuleProps {
  timeRange: string;
}

// 主要职责:
// - 项目指标数据获取和展示
// - 燃尽图SVG绘制
// - 数据可视化交互
// - 错误处理和加载状态
```

#### 2. StageOutputModule
```typescript
interface StageOutputModuleProps {
  timeRange: string;
}

// 主要职责:
// - 阶段数据管理
// - 横向滚动导航
// - 文档预览功能
// - 展开收起控制
```

#### 3. DigitalWorkerStatusModule
```typescript
interface DigitalWorkerStatusModuleProps {
  timeRange: string;
}

// 主要职责:
// - 员工状态数据管理
// - 表格展示和交互
// - 筛选排序逻辑
// - @mention功能
```

### 基础组件层 (Base Component Layer)
- **Card**: 统一的卡片容器组件
- **AccessibleButton**: 可访问的按钮组件
- **Spinner**: 加载状态指示器
- **ErrorBoundary**: 错误边界处理

## 🔄 数据流设计

### 数据流向图
```
用户交互 → 主容器状态 → 子模块props → 数据服务 → UI更新
    ↑                                            ↓
错误处理 ← 错误边界 ← 组件错误 ← 数据加载失败
```

### 状态管理模式

#### 主容器状态
```typescript
interface DashboardState {
  timeRange: string;                    // 时间范围
  customDateRange: {                    // 自定义日期范围
    start: string;
    end: string;
  };
  isCustomRangeVisible: boolean;        // 自定义范围显示状态
  lastUpdated: Date;                    // 最后更新时间
}
```

#### 模块级状态
每个模块维护自己的局部状态：
- **加载状态** (loading)
- **错误状态** (error)
- **数据状态** (data)
- **交互状态** (ui state)

### 事件通信机制

#### 全局事件
```typescript
// 数据刷新事件
window.dispatchEvent(new CustomEvent('dashboard-refresh'));

// 各模块监听刷新事件
window.addEventListener('dashboard-refresh', handleRefresh);
```

#### Props向下传递
```typescript
// 时间范围变更自动传递给所有子模块
<ProjectProgressModule timeRange={timeRange} />
<StageOutputModule timeRange={timeRange} />
<DigitalWorkerStatusModule timeRange={timeRange} />
```

## 🎨 UI设计模式

### 卡片网格布局
```css
.mx_CardGrid--large {
  grid-template-columns: repeat(auto-fit, minmax(480px, 1fr));
  gap: var(--workspace-space-6);
  max-width: 1400px;
}
```

### 响应式设计策略
- **桌面端** (>1024px): 3列网格布局
- **平板端** (768px-1024px): 2列网格布局
- **移动端** (<768px): 单列垂直布局

### 视觉层次设计
1. **主要信息**: 使用大字体和高对比度
2. **次要信息**: 使用中等字体和中等对比度
3. **辅助信息**: 使用小字体和低对比度

## 🔧 技术架构

### 前端技术栈
```typescript
// React生态
React 18+                 // 主框架
TypeScript 4.8+          // 类型系统
React Hooks             // 状态管理

// 样式系统
PostCSS                 // CSS预处理
CSS Module              // 样式隔离
CSS Variables           // 主题系统

// 数据可视化
SVG                     // 矢量图形
CSS Animation           // 动画效果
Canvas API              // 高性能绘制
```

### 服务层设计
```typescript
// 数据服务抽象
interface DataService {
  generateProjectMetrics(): ProjectMetrics;
  generateStageOutputs(): StageOutput[];
  generateDigitalWorkers(): DigitalWorker[];
}

// 错误处理服务
interface ErrorHandler {
  hasError: boolean;
  errorMessage: string;
  clearError(): void;
  wrapAsync<T>(fn: () => Promise<T>): Promise<T>;
}
```

### Hook封装模式
```typescript
// 错误处理Hook
const useErrorHandler = () => {
  const [hasError, setHasError] = useState(false);
  const [errorMessage, setErrorMessage] = useState('');

  const clearError = useCallback(() => {
    setHasError(false);
    setErrorMessage('');
  }, []);

  const wrapAsync = useCallback(async <T>(fn: () => Promise<T>) => {
    try {
      clearError();
      return await fn();
    } catch (error) {
      setHasError(true);
      setErrorMessage(error.message);
      throw error;
    }
  }, [clearError]);

  return { hasError, errorMessage, clearError, wrapAsync };
};
```

## 📊 数据模型设计

### 项目指标模型
```typescript
interface ProjectMetrics {
  totalTasks: number;           // 总任务数
  completedTasks: number;       // 完成任务数
  completionRate: number;       // 完成率
  weeklyNewTasks: number;       // 本周新增任务
  burndownData: BurndownPoint[]; // 燃尽图数据
}

interface BurndownPoint {
  date: string;                 // 日期
  ideal: number;                // 理想剩余任务
  actual: number;               // 实际剩余任务
}
```

### 阶段产出模型
```typescript
interface StageOutput {
  id: string;                   // 阶段ID
  name: string;                 // 阶段名称
  progress: number;             // 进度百分比
  documents: DocumentItem[];    // 文档列表
}

interface DocumentItem {
  id: string;                   // 文档ID
  name: string;                 // 文档名称
  type: DocumentType;           // 文档类型
  size: string;                 // 文件大小
  lastModified: Date;           // 最后修改时间
  author: string;               // 作者
}
```

### 员工状态模型
```typescript
interface DigitalWorker {
  id: string;                   // 员工ID
  name: string;                 // 姓名
  role: string;                 // 角色
  avatar: string;               // 头像
  status: WorkerStatus;         // 工作状态
  currentTask: string;          // 当前任务
  lastActive: Date;             // 最后活跃时间
}

type WorkerStatus = 'idle' | 'working' | 'blocked';
```

## 🚀 性能优化策略

### 组件优化
```typescript
// 使用React.memo优化渲染
const ProjectProgressModule = React.memo<ProjectProgressModuleProps>(
  ({ timeRange }) => {
    // 组件实现
  },
  (prevProps, nextProps) => prevProps.timeRange === nextProps.timeRange
);

// 使用useMemo缓存计算结果
const processedData = useMemo(() => {
  return expensiveDataProcessing(rawData);
}, [rawData]);

// 使用useCallback缓存函数引用
const handleRefresh = useCallback(() => {
  setLastUpdated(new Date());
}, []);
```

### 渲染优化
```typescript
// 虚拟滚动优化大量数据
const VirtualizedList = () => {
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={50}
      itemData={items}
    >
      {Row}
    </FixedSizeList>
  );
};

// 懒加载优化初始渲染
const LazyModule = lazy(() => import('./ExpensiveModule'));
```

### 样式性能优化
```css
/* 启用硬件加速 */
.mx_EfficiencyDashboard_content {
  will-change: scroll-position;
  transform: translateZ(0);
}

/* 使用contain优化渲染边界 */
.mx_EfficiencyDashboard_content {
  contain: layout style;
}

/* 优化动画性能 */
.mx_Card {
  transition: transform var(--workspace-duration-normal) var(--workspace-ease-in-out);
}
```

## 🔌 扩展性设计

### 插件化架构
```typescript
// 模块注册系统
interface DashboardModule {
  id: string;
  name: string;
  component: React.ComponentType<any>;
  props?: Record<string, any>;
}

const moduleRegistry = new Map<string, DashboardModule>();

// 注册新模块
moduleRegistry.set('custom-analytics', {
  id: 'custom-analytics',
  name: '自定义分析',
  component: CustomAnalyticsModule,
  props: { dataSource: 'api' }
});
```

### 主题系统扩展
```typescript
// 主题配置接口
interface ThemeConfig {
  colors: ColorPalette;
  typography: TypographyScale;
  spacing: SpacingScale;
  shadows: ShadowSystem;
}

// 动态主题切换
const useTheme = () => {
  const [theme, setTheme] = useState<ThemeConfig>(defaultTheme);

  const applyTheme = useCallback((newTheme: ThemeConfig) => {
    // 动态更新CSS变量
    Object.entries(newTheme.colors).forEach(([key, value]) => {
      document.documentElement.style.setProperty(`--color-${key}`, value);
    });
    setTheme(newTheme);
  }, []);

  return { theme, applyTheme };
};
```

### API扩展性
```typescript
// 数据源抽象
interface DataSource {
  getProjectMetrics(timeRange: string): Promise<ProjectMetrics>;
  getStageOutputs(timeRange: string): Promise<StageOutput[]>;
  getDigitalWorkers(timeRange: string): Promise<DigitalWorker[]>;
}

// 多数据源支持
class CompositeDataSource implements DataSource {
  constructor(private sources: DataSource[]) {}

  async getProjectMetrics(timeRange: string): Promise<ProjectMetrics> {
    // 聚合多个数据源
    const results = await Promise.all(
      this.sources.map(source => source.getProjectMetrics(timeRange))
    );
    return this.mergeMetrics(results);
  }
}
```

## 🛡️ 错误处理架构

### 分层错误处理
```typescript
// 全局错误边界
class GlobalErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // 记录错误日志
    console.error('Dashboard Error:', error, errorInfo);

    // 发送错误报告
    this.reportError(error, errorInfo);
  }
}

// 模块级错误处理
const ModuleErrorHandler = ({ children }: { children: React.ReactNode }) => {
  return (
    <ErrorBoundary
      fallback={<ModuleErrorFallback />}
      onError={(error) => console.error('Module Error:', error)}
    >
      {children}
    </ErrorBoundary>
  );
};
```

### 错误恢复策略
```typescript
// 自动重试机制
const useRetryableRequest = <T>(
  requestFn: () => Promise<T>,
  maxRetries: number = 3
) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async (retryCount = 0) => {
    try {
      setLoading(true);
      setError(null);
      const result = await requestFn();
      setData(result);
    } catch (err) {
      if (retryCount < maxRetries) {
        // 指数退避重试
        setTimeout(() => execute(retryCount + 1), 1000 * Math.pow(2, retryCount));
      } else {
        setError(err as Error);
      }
    } finally {
      setLoading(false);
    }
  }, [requestFn, maxRetries]);

  return { data, loading, error, execute };
};
```

这个架构设计确保了效率看板功能的**可维护性**、**可扩展性**和**高性能**，同时保持了良好的用户体验和开发体验。