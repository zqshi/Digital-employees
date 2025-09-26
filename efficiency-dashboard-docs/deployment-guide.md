# 效率看板 - 部署和配置指南

## 📦 环境要求

### 基础环境
- **Node.js**: 16.0.0 或更高版本
- **npm**: 8.0.0 或更高版本 (或 yarn 1.22.0+)
- **React**: 18.0.0 或更高版本
- **TypeScript**: 4.8.0 或更高版本

### 浏览器支持
- Chrome 88+
- Firefox 85+
- Safari 14+
- Edge 88+

### 开发工具
- VS Code (推荐)
- React Developer Tools
- TypeScript 语言服务

## 🚀 快速部署

### 1. 作为独立应用部署

#### 创建新项目
```bash
# 使用 Create React App
npx create-react-app efficiency-dashboard --template typescript
cd efficiency-dashboard

# 安装依赖
npm install classnames

# 复制效率看板组件文件
mkdir -p src/components/views/dashboard
mkdir -p src/components/views/elements
mkdir -p src/components/structures
mkdir -p src/services
mkdir -p src/types
mkdir -p src/hooks
```

#### 项目结构
```
src/
├── components/
│   ├── structures/
│   │   └── ErrorBoundary.tsx
│   └── views/
│       ├── dashboard/
│       │   ├── EfficiencyDashboard.tsx
│       │   ├── ProjectProgressModule.tsx
│       │   ├── StageOutputModule.tsx
│       │   ├── DigitalWorkerStatusModule.tsx
│       │   └── _EfficiencyDashboard.pcss
│       └── elements/
│           ├── AccessibleButton.tsx
│           ├── Card.tsx
│           ├── Spinner.tsx
│           ├── _Card.pcss
│           └── _design-tokens.pcss
├── services/
│   └── DataService.ts
├── types/
│   └── shared.ts
├── hooks/
│   └── useErrorHandler.ts
└── languageHandler.ts
```

#### 配置文件设置

**package.json 依赖配置**
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^4.8.0",
    "classnames": "^2.3.2"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "postcss": "^8.4.0",
    "postcss-preset-env": "^7.8.0"
  }
}
```

**postcss.config.js**
```javascript
module.exports = {
  plugins: [
    require('postcss-preset-env')({
      stage: 3,
      features: {
        'custom-properties': true,
        'nesting-rules': true
      }
    })
  ]
}
```

**tsconfig.json 配置**
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```

### 2. 集成到现有React项目

#### 安装步骤
```bash
# 1. 安装必要依赖
npm install classnames

# 2. 复制组件文件到项目中
# 将 dashboard/ 目录复制到 src/components/views/
# 将相关的类型定义和服务文件复制到对应目录

# 3. 配置CSS预处理
npm install postcss postcss-preset-env --save-dev
```

#### 在项目中使用
```typescript
// App.tsx
import React from 'react';
import EfficiencyDashboard from './components/views/dashboard/EfficiencyDashboard';
import './components/views/dashboard/_EfficiencyDashboard.pcss';

function App() {
  return (
    <div className="App">
      <EfficiencyDashboard />
    </div>
  );
}

export default App;
```

### 3. 作为Element Web扩展部署

#### 集成到Element Web
```bash
# 1. 克隆Element Web项目
git clone https://github.com/vector-im/element-web.git
cd element-web

# 2. 安装依赖
npm install

# 3. 复制效率看板文件
# 将dashboard组件文件复制到 src/components/views/dashboard/
# 将相关类型和服务文件复制到对应目录

# 4. 注册新的RightPanel阶段
# 在 src/stores/right-panel/RightPanelStorePhases.ts 中添加:
# EfficiencyDashboard = "EfficiencyDashboard",

# 5. 集成到RightPanel组件
# 在 src/components/structures/RightPanel.tsx 中添加渲染逻辑

# 6. 添加入口按钮
# 在 src/components/views/rooms/RoomHeader/RoomHeader.tsx 中添加按钮
```

## ⚙️ 配置选项

### 数据源配置

#### Mock数据模式 (默认)
```typescript
// src/services/DataService.ts
export class DataService {
  static readonly USE_MOCK_DATA = true;

  static generateProjectMetrics(): ProjectMetrics {
    // Mock数据生成逻辑
  }

  static generateStageOutputs(): StageOutput[] {
    // Mock数据生成逻辑
  }

  static generateDigitalWorkers(): DigitalWorker[] {
    // Mock数据生成逻辑
  }
}
```

#### 真实API模式
```typescript
// src/services/DataService.ts
export class DataService {
  static readonly USE_MOCK_DATA = false;
  static readonly API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3001';

  static async getProjectMetrics(timeRange: string): Promise<ProjectMetrics> {
    const response = await fetch(`${this.API_BASE_URL}/api/metrics?timeRange=${timeRange}`);
    if (!response.ok) {
      throw new Error('Failed to fetch project metrics');
    }
    return response.json();
  }

  static async getStageOutputs(timeRange: string): Promise<StageOutput[]> {
    const response = await fetch(`${this.API_BASE_URL}/api/stages?timeRange=${timeRange}`);
    if (!response.ok) {
      throw new Error('Failed to fetch stage outputs');
    }
    return response.json();
  }

  static async getDigitalWorkers(): Promise<DigitalWorker[]> {
    const response = await fetch(`${this.API_BASE_URL}/api/workers`);
    if (!response.ok) {
      throw new Error('Failed to fetch digital workers');
    }
    return response.json();
  }
}
```

### 主题配置

#### CSS变量自定义
```css
/* src/theme/custom-theme.css */
:root {
  /* 品牌色彩 */
  --workspace-brand-primary: #1976d2;
  --workspace-brand-secondary: #42a5f5;
  --workspace-brand-primary-hover: #1565c0;

  /* 状态色彩 */
  --workspace-success: #4caf50;
  --workspace-info: #2196f3;
  --workspace-warning: #ff9800;
  --workspace-error: #f44336;

  /* 中性色彩 */
  --workspace-neutral-50: #fafafa;
  --workspace-neutral-100: #f5f5f5;
  --workspace-neutral-200: #eeeeee;
  --workspace-neutral-500: #9e9e9e;

  /* 间距系统 */
  --workspace-space-1: 4px;
  --workspace-space-2: 8px;
  --workspace-space-3: 12px;
  --workspace-space-4: 16px;
  --workspace-space-6: 24px;

  /* 字体系统 */
  --workspace-text-xs: 0.75rem;
  --workspace-text-sm: 0.875rem;
  --workspace-text-base: 1rem;
  --workspace-text-lg: 1.125rem;
  --workspace-text-xl: 1.25rem;
  --workspace-text-2xl: 1.5rem;
  --workspace-text-3xl: 1.875rem;

  /* 圆角系统 */
  --workspace-radius-sm: 4px;
  --workspace-radius-md: 6px;
  --workspace-radius-lg: 8px;
  --workspace-radius-xl: 12px;

  /* 阴影系统 */
  --workspace-shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --workspace-shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --workspace-shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --workspace-shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}
```

#### 深色主题支持
```css
/* src/theme/dark-theme.css */
@media (prefers-color-scheme: dark) {
  :root {
    --workspace-neutral-50: #1a1a1a;
    --workspace-neutral-100: #262626;
    --workspace-neutral-200: #404040;
    --workspace-neutral-500: #737373;

    --cpd-color-bg-canvas-default: #1a1a1a;
    --cpd-color-bg-subtle-primary: #262626;
    --cpd-color-bg-subtle-secondary: #404040;
    --cpd-color-text-primary: #ffffff;
    --cpd-color-text-secondary: #a3a3a3;
  }
}
```

### 国际化配置

#### 语言文件
```typescript
// src/i18n/strings/zh_CN.json
{
  "效率看板": "效率看板",
  "项目进度跟踪与团队状态监控": "项目进度跟踪与团队状态监控",
  "项目进度总览": "项目进度总览",
  "燃尽图和关键指标展示": "燃尽图和关键指标展示",
  "阶段产出物": "阶段产出物",
  "按开发阶段查看文档产出": "按开发阶段查看文档产出",
  "数字员工状态": "数字员工状态",
  "团队成员工作状态实时监控": "团队成员工作状态实时监控",
  "本周": "本周",
  "本月": "本月",
  "本季度": "本季度",
  "自定义范围...": "自定义范围...",
  "刷新数据": "刷新数据",
  "最后更新": "最后更新"
}

// src/i18n/strings/en_EN.json
{
  "效率看板": "Efficiency Dashboard",
  "项目进度跟踪与团队状态监控": "Project Progress Tracking & Team Status Monitoring",
  "项目进度总览": "Project Progress Overview",
  "燃尽图和关键指标展示": "Burndown Chart and Key Metrics",
  "阶段产出物": "Stage Outputs",
  "按开发阶段查看文档产出": "View Documents by Development Stage",
  "数字员工状态": "Digital Worker Status",
  "团队成员工作状态实时监控": "Real-time Team Member Status Monitoring",
  "本周": "This Week",
  "本月": "This Month",
  "本季度": "This Quarter",
  "自定义范围...": "Custom Range...",
  "刷新数据": "Refresh Data",
  "最后更新": "Last Updated"
}
```

#### 语言处理器
```typescript
// src/languageHandler.ts
const translations: Record<string, Record<string, string>> = {
  'zh_CN': require('./i18n/strings/zh_CN.json'),
  'en_EN': require('./i18n/strings/en_EN.json')
};

let currentLanguage = 'zh_CN';

export const _t = (key: string, substitutions?: Record<string, any>): string => {
  const translation = translations[currentLanguage]?.[key] || key;

  if (!substitutions) {
    return translation;
  }

  return Object.entries(substitutions).reduce(
    (result, [placeholder, value]) =>
      result.replace(new RegExp(`%\\(${placeholder}\\)s`, 'g'), String(value)),
    translation
  );
};

export const setLanguage = (language: string): void => {
  if (translations[language]) {
    currentLanguage = language;
  }
};
```

## 🔧 构建配置

### 开发环境
```bash
# 启动开发服务器
npm start

# 启动时带环境变量
REACT_APP_API_URL=http://localhost:3001 npm start

# 启动时指定端口
PORT=3001 npm start
```

### 生产构建
```bash
# 构建生产版本
npm run build

# 构建并分析包大小
npm install --save-dev webpack-bundle-analyzer
npm run build && npx webpack-bundle-analyzer build/static/js/*.js
```

### Docker部署
```dockerfile
# Dockerfile
FROM node:16-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api {
            proxy_pass http://backend:3001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: .
    ports:
      - "80:80"
    depends_on:
      - backend
    environment:
      - REACT_APP_API_URL=http://localhost:3001

  backend:
    image: your-api-server:latest
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/dbname

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=dbname
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 📊 性能优化

### 代码分割
```typescript
// 懒加载大型组件
import { lazy, Suspense } from 'react';

const EfficiencyDashboard = lazy(() => import('./components/views/dashboard/EfficiencyDashboard'));

function App() {
  return (
    <Suspense fallback={<div>Loading Dashboard...</div>}>
      <EfficiencyDashboard />
    </Suspense>
  );
}
```

### 构建优化
```javascript
// webpack.config.js (如果使用自定义配置)
const path = require('path');

module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        dashboard: {
          test: /[\\/]src[\\/]components[\\/]views[\\/]dashboard[\\/]/,
          name: 'dashboard',
          chunks: 'all',
        }
      }
    }
  }
};
```

### CDN配置
```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8" />
  <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />

  <!-- 预加载关键字体 -->
  <link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin>

  <!-- DNS预解析 -->
  <link rel="dns-prefetch" href="//api.yourdomain.com">

  <title>效率看板</title>
</head>
<body>
  <noscript>您需要启用JavaScript来运行此应用。</noscript>
  <div id="root"></div>
</body>
</html>
```

## 🔍 监控和日志

### 错误监控
```typescript
// src/utils/errorReporting.ts
export const reportError = (error: Error, context?: Record<string, any>) => {
  // 可以集成 Sentry, LogRocket 等监控服务
  console.error('Dashboard Error:', error, context);

  // 发送到监控服务
  if (process.env.NODE_ENV === 'production') {
    // Sentry.captureException(error, { extra: context });
  }
};

// 在ErrorBoundary中使用
class DashboardErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    reportError(error, {
      component: 'EfficiencyDashboard',
      errorInfo
    });
  }
}
```

### 性能监控
```typescript
// src/utils/performance.ts
export const measurePerformance = (name: string, fn: () => void) => {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);
  performance.measure(name, `${name}-start`, `${name}-end`);

  const measure = performance.getEntriesByName(name)[0];
  console.log(`${name} took ${measure.duration}ms`);
};

// 使用示例
measurePerformance('dashboard-render', () => {
  // 渲染逻辑
});
```

## 🧪 测试配置

### 单元测试
```bash
# 安装测试依赖
npm install --save-dev @testing-library/react @testing-library/jest-dom jest-environment-jsdom
```

```typescript
// src/components/views/dashboard/__tests__/EfficiencyDashboard.test.tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import EfficiencyDashboard from '../EfficiencyDashboard';

describe('EfficiencyDashboard', () => {
  test('renders dashboard title', () => {
    render(<EfficiencyDashboard />);
    expect(screen.getByText('效率看板')).toBeInTheDocument();
  });

  test('displays all three modules', () => {
    render(<EfficiencyDashboard />);
    expect(screen.getByText('项目进度总览')).toBeInTheDocument();
    expect(screen.getByText('阶段产出物')).toBeInTheDocument();
    expect(screen.getByText('数字员工状态')).toBeInTheDocument();
  });
});
```

### E2E测试
```bash
# 安装Playwright
npm install --save-dev @playwright/test
```

```typescript
// e2e/dashboard.spec.ts
import { test, expect } from '@playwright/test';

test('efficiency dashboard loads correctly', async ({ page }) => {
  await page.goto('/');

  // 检查页面标题
  await expect(page.locator('h2')).toContainText('效率看板');

  // 检查三个模块都存在
  await expect(page.locator('.mx_Card')).toHaveCount(3);

  // 测试时间范围选择器
  await page.selectOption('select', '本月');
  await expect(page.locator('select')).toHaveValue('本月');
});
```

这个部署指南提供了从开发到生产的完整部署流程，确保效率看板能够在各种环境中稳定运行。