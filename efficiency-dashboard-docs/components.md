# 效率看板 - 组件详细说明文档

## 📋 组件总览

效率看板由4个核心组件构成，每个组件负责特定的功能领域，共同构建完整的项目管理可视化界面。

| 组件名称 | 文件名 | 代码量 | 主要功能 |
|---------|--------|--------|----------|
| EfficiencyDashboard | EfficiencyDashboard.tsx | 196行 | 主容器和状态管理 |
| ProjectProgressModule | ProjectProgressModule.tsx | 249行 | 项目进度可视化 |
| StageOutputModule | StageOutputModule.tsx | 277行 | 阶段产出管理 |
| DigitalWorkerStatusModule | DigitalWorkerStatusModule.tsx | 291行 | 员工状态监控 |

## 🏗️ 主容器组件

### EfficiencyDashboard.tsx

**职责概述**
- 整体布局和状态管理
- 时间范围控制逻辑
- 自动刷新机制
- 子组件协调

**组件接口**
```typescript
interface EfficiencyDashboardProps {}

interface DashboardState {
  timeRange: string;
  customDateRange: {
    start: string;
    end: string;
  };
  isCustomRangeVisible: boolean;
  lastUpdated: Date;
}
```

**核心功能**

#### 1. 时间范围控制
```typescript
const handleTimeRangeChange = (value: string) => {
  setTimeRange(value);
  if (value === '自定义范围...') {
    setIsCustomRangeVisible(true);
  } else {
    setIsCustomRangeVisible(false);
  }
};
```

#### 2. 自动刷新机制
```typescript
useEffect(() => {
  const interval = setInterval(() => {
    setLastUpdated(new Date());
  }, 60000); // 每分钟刷新一次

  return () => clearInterval(interval);
}, []);
```

#### 3. 手动刷新控制
```typescript
const refreshData = () => {
  setLastUpdated(new Date());
  // 触发所有子组件数据刷新
  window.dispatchEvent(new CustomEvent('dashboard-refresh'));
};
```

**UI结构**
```jsx
<div className="mx_EfficiencyDashboard">
  {/* 头部控制区 */}
  <div className="mx_EfficiencyDashboard_header">
    <div className="mx_EfficiencyDashboard_title">
      <h2>效率看板</h2>
      <p>项目进度跟踪与团队状态监控</p>
    </div>
    <div className="mx_EfficiencyDashboard_controls">
      {/* 时间选择器 */}
      {/* 刷新按钮 */}
      {/* 最后更新时间 */}
    </div>
  </div>

  {/* 内容区域 */}
  <div className="mx_EfficiencyDashboard_content">
    <div className="mx_CardGrid mx_CardGrid--large">
      {/* 三个功能模块 */}
    </div>
  </div>
</div>
```

## 📊 项目进度模块

### ProjectProgressModule.tsx

**职责概述**
- 项目关键指标展示
- 燃尽图数据可视化
- 进度趋势分析
- 实时数据更新

**组件接口**
```typescript
interface ProjectProgressModuleProps {
  timeRange: string;
}

interface ProjectProgressState {
  metrics: ProjectMetrics | null;
  loading: boolean;
  error: Error | null;
}
```

**数据模型**
```typescript
interface ProjectMetrics {
  totalTasks: number;           // 总任务数
  completedTasks: number;       // 完成任务数
  completionRate: number;       // 完成率
  weeklyNewTasks: number;       // 本周新增任务
  burndownData: BurndownPoint[]; // 燃尽图数据点
}

interface BurndownPoint {
  date: string;     // 日期 (YYYY-MM-DD)
  ideal: number;    // 理想剩余任务数
  actual: number;   // 实际剩余任务数
}
```

**核心功能**

#### 1. 指标卡片组件
```typescript
const MetricCard: React.FC<{
  title: string;
  value: number;
  unit?: string;
  progress?: number;
}> = ({ title, value, unit, progress }) => (
  <div className="mx_MetricCard">
    <h4 className="mx_MetricCard_title">{title}</h4>
    <div className="mx_MetricCard_value">
      {value}
      {unit && <span className="mx_MetricCard_unit">{unit}</span>}
    </div>
    {progress !== undefined && (
      <div className="mx_MetricCard_progress">
        <div className="mx_MetricCard_progressBar">
          <div
            className="mx_MetricCard_progressFill"
            style={{ width: `${progress}%` }}
          />
        </div>
      </div>
    )}
  </div>
);
```

#### 2. 燃尽图SVG组件
```typescript
const BurndownChart: React.FC<{
  data: BurndownPoint[];
  width: number;
  height: number;
}> = ({ data, width, height }) => {
  const margin = { top: 20, right: 30, bottom: 40, left: 40 };
  const chartWidth = width - margin.left - margin.right;
  const chartHeight = height - margin.top - margin.bottom;

  // SVG路径计算
  const idealPath = useMemo(() => {
    return data.map((point, index) => {
      const x = (index / (data.length - 1)) * chartWidth;
      const y = ((1 - point.ideal / maxTasks) * chartHeight);
      return `${index === 0 ? 'M' : 'L'} ${x} ${y}`;
    }).join(' ');
  }, [data, chartWidth, chartHeight]);

  const actualPath = useMemo(() => {
    return data.map((point, index) => {
      const x = (index / (data.length - 1)) * chartWidth;
      const y = ((1 - point.actual / maxTasks) * chartHeight);
      return `${index === 0 ? 'M' : 'L'} ${x} ${y}`;
    }).join(' ');
  }, [data, chartWidth, chartHeight]);

  return (
    <svg width={width} height={height} className="mx_BurndownChart_svg">
      <g transform={`translate(${margin.left},${margin.top})`}>
        {/* 网格线 */}
        {/* 理想线 (虚线) */}
        <path
          d={idealPath}
          stroke="#4CAF50"
          strokeWidth="2"
          strokeDasharray="5,5"
          fill="none"
        />
        {/* 实际线 (实线) */}
        <path
          d={actualPath}
          stroke="#2196F3"
          strokeWidth="2"
          fill="none"
        />
        {/* 数据点 */}
        {data.map((point, index) => (
          <circle
            key={index}
            cx={(index / (data.length - 1)) * chartWidth}
            cy={((1 - point.actual / maxTasks) * chartHeight)}
            r="3"
            fill="#2196F3"
            className="mx_BurndownChart_point"
          />
        ))}
      </g>
    </svg>
  );
};
```

#### 3. 数据加载和错误处理
```typescript
useEffect(() => {
  const loadData = () => {
    setLoading(true);
    timeoutRef.current = setTimeout(() => {
      try {
        const newMetrics = DataService.generateProjectMetrics();
        setMetrics(newMetrics);
        setError(null);
      } catch (err) {
        setError(err as Error);
        setMetrics(null);
      } finally {
        setLoading(false);
      }
    }, 500); // 模拟API延迟
  };

  loadData();

  // 监听全局刷新事件
  const handleRefresh = () => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    loadData();
  };

  window.addEventListener('dashboard-refresh', handleRefresh);
  return () => {
    window.removeEventListener('dashboard-refresh', handleRefresh);
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
  };
}, [timeRange]);
```

## 📄 阶段产出模块

### StageOutputModule.tsx

**职责概述**
- 开发阶段可视化展示
- 文档聚合和预览
- 横向滚动导航
- 阶段进度跟踪

**组件接口**
```typescript
interface StageOutputModuleProps {
  timeRange: string;
}

interface StageOutputState {
  stageOutputs: StageOutput[];
  selectedDocument: DocumentItem | null;
  expandedStage: string | null;
  loading: boolean;
}
```

**数据模型**
```typescript
interface StageOutput {
  id: string;          // 阶段唯一标识
  name: string;        // 阶段名称
  progress: number;    // 进度百分比 (0-100)
  documents: DocumentItem[]; // 该阶段的文档列表
}

interface DocumentItem {
  id: string;          // 文档ID
  name: string;        // 文档名称
  type: DocumentType;  // 文档类型
  size: string;        // 文件大小
  lastModified: Date;  // 最后修改时间
  author: string;      // 作者
  url?: string;        // 文档URL
}

type DocumentType = 'pdf' | 'word' | 'excel' | 'markdown' | 'image';
```

**核心功能**

#### 1. 阶段卡片组件
```typescript
const StageCard: React.FC<{
  stage: StageOutput;
  isExpanded: boolean;
  onToggleExpand: () => void;
  onDocumentClick: (doc: DocumentItem) => void;
}> = ({ stage, isExpanded, onToggleExpand, onDocumentClick }) => (
  <div className="mx_StageCard">
    <div className="mx_StageCard_header">
      <h4>{stage.name}</h4>
      <span className="mx_StageCard_badge">
        {stage.documents.length} 个文档
      </span>
    </div>

    {/* 进度条 */}
    <div className="mx_StageCard_progress">
      <div className="mx_StageCard_progressBar">
        <div
          className="mx_StageCard_progressFill"
          style={{ width: `${stage.progress}%` }}
        />
      </div>
      <span>{stage.progress}% 完成</span>
    </div>

    {/* 文档列表 */}
    <div className="mx_StageCard_documents">
      {stage.documents
        .slice(0, isExpanded ? undefined : 3)
        .map(doc => (
          <div
            key={doc.id}
            className="mx_StageCard_document"
            onClick={() => onDocumentClick(doc)}
          >
            <span className="mx_StageCard_documentIcon">
              {getDocumentIcon(doc.type)}
            </span>
            <div className="mx_StageCard_documentInfo">
              <div className="mx_StageCard_documentName">
                {doc.name}
              </div>
              <div className="mx_StageCard_documentMeta">
                <span>{doc.size}</span>
                <span>{doc.author}</span>
              </div>
            </div>
          </div>
        ))
      }
    </div>

    {/* 展开/收起按钮 */}
    {stage.documents.length > 3 && (
      <div className="mx_StageCard_footer">
        <AccessibleButton
          className="mx_StageCard_viewAllBtn"
          onClick={onToggleExpand}
        >
          {isExpanded ? '收起' : `查看全部 (${stage.documents.length})`}
        </AccessibleButton>
      </div>
    )}

    {/* 连接线装饰 */}
    <div className="mx_StageCard_connector">
      <div className="mx_StageCard_connectorLine" />
      <span className="mx_StageCard_connectorArrow">→</span>
    </div>
  </div>
);
```

#### 2. 横向滚动控制
```typescript
const scrollLeft = useCallback(() => {
  if (scrollRef.current) {
    scrollRef.current.scrollBy({
      left: -280, // 卡片宽度
      behavior: 'smooth'
    });
  }
}, []);

const scrollRight = useCallback(() => {
  if (scrollRef.current) {
    scrollRef.current.scrollBy({
      left: 280,
      behavior: 'smooth'
    });
  }
}, []);
```

#### 3. 文档图标映射
```typescript
const getDocumentIcon = (type: DocumentType): string => {
  const iconMap: Record<DocumentType, string> = {
    'pdf': '📄',
    'word': '📝',
    'excel': '📊',
    'markdown': '📋',
    'image': '🖼️'
  };
  return iconMap[type] || '📄';
};
```

## 👥 数字员工状态模块

### DigitalWorkerStatusModule.tsx

**职责概述**
- 员工工作状态实时监控
- 任务分配情况展示
- 状态统计和筛选
- @mention功能支持

**组件接口**
```typescript
interface DigitalWorkerStatusModuleProps {
  timeRange: string;
}

interface WorkerStatusState {
  workers: DigitalWorker[];
  filteredWorkers: DigitalWorker[];
  statusFilter: WorkerStatus | 'all';
  sortBy: 'name' | 'role' | 'lastActive';
  sortOrder: 'asc' | 'desc';
  loading: boolean;
}
```

**数据模型**
```typescript
interface DigitalWorker {
  id: string;           // 员工唯一标识
  name: string;         // 员工姓名
  role: string;         // 职位角色
  avatar: string;       // 头像emoji
  status: WorkerStatus; // 当前工作状态
  currentTask: string;  // 正在执行的任务
  lastActive: Date;     // 最后活跃时间
}

type WorkerStatus = 'idle' | 'working' | 'blocked';
```

**核心功能**

#### 1. 状态统计面板
```typescript
const WorkerStats: React.FC<{ workers: DigitalWorker[] }> = ({ workers }) => {
  const stats = useMemo(() => {
    const totalWorkers = workers.length;
    const workingCount = workers.filter(w => w.status === 'working').length;
    const idleCount = workers.filter(w => w.status === 'idle').length;
    const blockedCount = workers.filter(w => w.status === 'blocked').length;

    return { totalWorkers, workingCount, idleCount, blockedCount };
  }, [workers]);

  return (
    <div className="mx_WorkerStats">
      <div className="mx_WorkerStats_item">
        <div className="mx_WorkerStats_count">{stats.totalWorkers}</div>
        <div className="mx_WorkerStats_label">总员工</div>
      </div>
      <div className="mx_WorkerStats_item mx_WorkerStats_working">
        <div className="mx_WorkerStats_count">{stats.workingCount}</div>
        <div className="mx_WorkerStats_label">工作中</div>
      </div>
      <div className="mx_WorkerStats_item mx_WorkerStats_idle">
        <div className="mx_WorkerStats_count">{stats.idleCount}</div>
        <div className="mx_WorkerStats_label">空闲</div>
      </div>
      <div className="mx_WorkerStats_item mx_WorkerStats_blocked">
        <div className="mx_WorkerStats_count">{stats.blockedCount}</div>
        <div className="mx_WorkerStats_label">阻塞</div>
      </div>
    </div>
  );
};
```

#### 2. 员工表格组件
```typescript
const WorkerTable: React.FC<{
  workers: DigitalWorker[];
  onMention: (worker: DigitalWorker) => void;
}> = ({ workers, onMention }) => (
  <div className="mx_WorkerTable">
    {/* 表格头部 */}
    <div className="mx_WorkerTable_header">
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_nameHeader">
        姓名
      </div>
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_roleHeader">
        角色
      </div>
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_statusHeader">
        状态
      </div>
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_taskHeader">
        当前任务
      </div>
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_timeHeader">
        最后活跃
      </div>
      <div className="mx_WorkerTable_headerCell mx_WorkerTable_actionHeader">
        操作
      </div>
    </div>

    {/* 表格内容 */}
    <div className="mx_WorkerTable_body">
      {workers.map(worker => (
        <div key={worker.id} className="mx_WorkerTable_row">
          <div className="mx_WorkerTable_cell mx_WorkerTable_nameCell">
            <div className="mx_WorkerInfo">
              <div className="mx_WorkerInfo_avatar">{worker.avatar}</div>
              <div className="mx_WorkerInfo_name">{worker.name}</div>
            </div>
          </div>
          <div className="mx_WorkerTable_cell mx_WorkerTable_roleCell">
            {worker.role}
          </div>
          <div className="mx_WorkerTable_cell mx_WorkerTable_statusCell">
            <div className={`mx_WorkerStatus mx_WorkerStatus_${worker.status}`}>
              <div className="mx_WorkerStatus_indicator" />
              <span className="mx_WorkerStatus_text">
                {getStatusText(worker.status)}
              </span>
            </div>
          </div>
          <div className="mx_WorkerTable_cell mx_WorkerTable_taskCell">
            <div className="mx_WorkerTask">
              {worker.currentTask || '无任务'}
            </div>
          </div>
          <div className="mx_WorkerTable_cell mx_WorkerTable_timeCell">
            <div className="mx_WorkerTime">
              {formatRelativeTime(worker.lastActive)}
            </div>
          </div>
          <div className="mx_WorkerTable_cell mx_WorkerTable_actionCell">
            <AccessibleButton
              className="mx_WorkerTable_mentionBtn"
              onClick={() => onMention(worker)}
            >
              @TA
            </AccessibleButton>
          </div>
        </div>
      ))}
    </div>
  </div>
);
```

#### 3. 筛选和排序逻辑
```typescript
// 筛选逻辑
const filteredWorkers = useMemo(() => {
  let result = workers;

  // 按状态筛选
  if (statusFilter !== 'all') {
    result = result.filter(worker => worker.status === statusFilter);
  }

  // 排序
  result.sort((a, b) => {
    let comparison = 0;

    switch (sortBy) {
      case 'name':
        comparison = a.name.localeCompare(b.name);
        break;
      case 'role':
        comparison = a.role.localeCompare(b.role);
        break;
      case 'lastActive':
        comparison = a.lastActive.getTime() - b.lastActive.getTime();
        break;
    }

    return sortOrder === 'asc' ? comparison : -comparison;
  });

  return result;
}, [workers, statusFilter, sortBy, sortOrder]);

// @mention功能
const handleMention = useCallback((worker: DigitalWorker) => {
  // 这里可以集成真实的mention功能
  console.log(`Mentioning ${worker.name}`);

  // 模拟发送mention
  const mentionText = `@${worker.name}`;

  // 可以集成到聊天系统或通知系统
  if (window.mentionCallback) {
    window.mentionCallback(mentionText);
  }
}, []);
```

#### 4. 状态指示器组件
```typescript
const StatusIndicator: React.FC<{ status: WorkerStatus }> = ({ status }) => {
  const statusConfig = {
    idle: { color: 'success', text: '空闲', icon: '🟢' },
    working: { color: 'info', text: '工作中', icon: '🔵' },
    blocked: { color: 'critical', text: '阻塞', icon: '🔴' }
  };

  const config = statusConfig[status];

  return (
    <div className={`mx_WorkerStatus mx_WorkerStatus_${status}`}>
      <div className="mx_WorkerStatus_indicator" />
      <span className="mx_WorkerStatus_text">{config.text}</span>
    </div>
  );
};
```

## 🔧 通用工具组件

### 错误边界组件
```typescript
const ModuleErrorBoundary: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <ErrorBoundary
      fallback={
        <div className="mx_Module_error">
          <p>模块加载失败，请刷新重试</p>
          <AccessibleButton onClick={() => window.location.reload()}>
            刷新页面
          </AccessibleButton>
        </div>
      }
    >
      {children}
    </ErrorBoundary>
  );
};
```

### 加载状态组件
```typescript
const LoadingSpinner: React.FC<{ message?: string }> = ({ message = '加载中...' }) => (
  <div className="mx_Module_loading">
    <Spinner />
    <p>{message}</p>
  </div>
);
```

### 时间格式化工具
```typescript
const formatRelativeTime = (date: Date): string => {
  const now = new Date();
  const diffInMinutes = Math.floor((now.getTime() - date.getTime()) / (1000 * 60));

  if (diffInMinutes < 1) return '刚刚';
  if (diffInMinutes < 60) return `${diffInMinutes}分钟前`;

  const diffInHours = Math.floor(diffInMinutes / 60);
  if (diffInHours < 24) return `${diffInHours}小时前`;

  const diffInDays = Math.floor(diffInHours / 24);
  return `${diffInDays}天前`;
};
```

这套组件设计确保了**模块化**、**可复用性**和**可维护性**，每个组件都有明确的职责分工和标准化的接口设计。