---
title: "UE Page Builder"
summary: "基于 Semi Design + Vite + React 的页面生成器。数据内联于附录中，输入需求即可生成可运行的前端页面。"
agent_created: true
---

# UE Page Builder

**用途**：根据用户需求（页面描述、截图、设计稿），生成基于 Semi Design 组件的可运行 Vite + React 页面。

**核心原则**：所有规则数据（CSS Token、组件映射、布局模板、图标名称、API 签名）直接在本 SKILL 附录中查找，避免凭经验猜测导致幻觉错误。

## 附录索引

| 附录 | 内容 | 数据文件 | 何时查阅 |
|------|------|----------|---------|
| 附录 A：Tokens | 颜色/间距/圆角/字号等 CSS 变量 | `data/tokens.json` | 生成 `:root` 和 CSS 样式时**必须读取该文件** |
| 附录 B：Components-Schema | Semi UI 组件注册表 | `data/components-schema.json` | 确定用什么组件时**必须读取该文件** |
| 附录 C：Layouts | 布局模板（骨架 + 业务页面） | `data/layouts.json` | 选择页面结构时**必须读取该文件** |
| 附录 D：常用图标速查 | 常用图标快速参考 | （内联） | 图标速查 |
| 附录 F：完整图标清单 | 523 个实际存在的 semi-icons | `data/icons.json` | **生成代码前必须验证图标名** |

> ⚠️ **数据文件读取指令（强制执行）**：
> - 需要查 CSS Token → 必须先 `Read` `data/tokens.json`
> - 需要查组件 Schema → 必须先 `Read` `data/components-schema.json`
> - 需要查布局模板 → 必须先 `Read` `data/layouts.json`
> - 需要查图标 → 必须先 `Read` `data/icons.json`（523 个图标，禁止凭语义猜测）
> - **禁止**在 SKILL.md 正文中凭记忆写组件 API、CSS 变量或图标名

## 人机协作流程（推荐）

为减少返工、提升还原度，采用"先确认方案，再生成代码"的协作模式。根据需求明确程度，分为两条路径：

### 路径 A：需求明确 → 快速生成

适用场景：用户提供了完整的页面描述、截图、或设计文档。

流程：
1. 执行**组件库能力边界检查**（见下方强制检查）
2. 直接读取三层文件 → 匹配布局和组件 → 生成项目 → 启动预览
3. 用户验收后迭代调整

### 路径 B：需求模糊 → 方案确认

适用场景：用户只有一句话需求，如"做一个产业详情页"。

流程：

**Round 0：组件库能力边界检查（强制，不可跳过）**
- 检查 `node_modules/@douyinfe/semi-ui/package.json` 中的**实际安装版本**（不是 package.json 中的声明版本）
- 检查 `node_modules/@douyinfe/semi-ui/lib/es/index.js` 中的导出清单，确认是否有与需求匹配的专用组件（如 AI 对话场景需确认 `AIChatDialogue`、`AIChatInput` 是否存在）
- 发现专用组件时，优先纳入方案；未发现的，才允许手写实现并标注原因
- **此步骤不可凭经验跳过**

**Round 0.5：AI 对话场景默认推荐（自动触发）**

当用户需求涉及"AI 对话"、"智能助手"、"AI 聊天"、"大模型应用"等关键词时，**默认**在方案中提出包含过程可视化的完整 AI 组件方案，供用户确认：

| 用户提到的关键词 | 默认推荐的子组件 | 展示效果 |
|-----------------|-----------------|---------|
| "展示思考过程"、"推理链"、"思维链"、"为什么这样回答" | `AIChatDialogue.Reasoning` | 可展开/折叠的思考过程区域 |
| "分步展示"、"执行流程"、"搜索步骤"、"任务进度"、"多步骤执行" | `AIChatDialogue.Step` | 分步展示 AI 执行任务流程（搜索/读取/生成） |
| "引用来源"、"参考材料"、"数据来源"、"信息出处" | `AIChatDialogue.Annotation` | 展示引用链接、标题、描述 |
| "完整的 AI 对话"、"专业 AI 助手"、"类 ChatGPT/Perplexity" | 以上三者组合 | 思考 → 步骤 → 回复 + 引用 |
| "联网搜索"、"知识库"、"模式切换"、"搜索增强"、"深度思考" | `AIChatInput.Configure` | 输入框底部配置区（Button开关 / Select选择 / RadioButton模式切换） |

**执行规则**：
1. 只要用户提到 AI 对话类需求（无论是否明确说"过程可视化"），**必须**主动提出包含 `Reasoning` + `Step` + `Annotation` 的完整方案
2. 同时**默认推荐** `AIChatInput` 配置区域：`Configure.Button`（联网搜索开关）、`Configure.Select`（知识库选择）、`Configure.RadioButton`（快速/思考模式切换）
3. 以"推荐方案"形式呈现，明确告知用户这三个子组件的能力及输入框配置能力
4. 询问用户："是否需要展示 AI 的思考过程和任务执行步骤？还是只需要简单的对话气泡？"
5. 询问用户："输入框是否需要配置区域（联网搜索、知识库选择、模式切换）？"
6. 用户确认后，在消息数据结构中注入 `reasoning` / `plan` / `annotations` 类型的 `ContentItem`，在输入框中注入 `renderConfigureArea` + `onConfigureChange`

> **原则**：宁可多一轮确认让用户选择简化方案，也不要默认只给最简单的对话气泡。

**Round 1：需求澄清 + 布局推荐**
- 根据用户描述，从 `layouts/index.json` 推荐 1-3 个最匹配的 baseLayouts / pageLayouts
- 说明推荐理由（如"该布局含顶部导航+侧边栏+内容区，适合管理后台"）
- 询问用户："是否确认使用该布局？或需要调整？"

**Round 2：组件清单确认（可选）**
- 如果页面复杂或用户有明确组件偏好，列出计划使用的 Semi 组件清单
- 格式：表格（组件名 | 用途 | 位置）
- 询问用户："组件清单是否满足需求？需要增删？"

**Round 3：生成代码**
- 确认方案后，进入 Step 3（确定项目模式）→ Step 4（生成文件）→ Step 5（启动预览）

> **原则**：宁可多一轮确认，也不要生成后整页推翻重来。

---

## 工作流

### Step 1: 读取三层数据文件

生成页面前**必须读取**以下文件（禁止凭记忆写组件名或变量值）：

- `data/tokens.json` → 提取 CSS 变量
- `data/components-schema.json` → 查询组件 API
- `data/layouts.json` → 选择布局模板
- `data/icons.json` → **验证图标名（生成含图标的代码前强制读取）**

> **警告**：附录 D（图标速查）内联于本文件，可直接查阅；A/B/C 数据文件已外置，必须通过 `Read` 工具读取。

### Step 2: 解析需求 → 组件边界检查 → 呈现方案 → 用户确认

**本步骤与人机协作流程中的 Round 0~3 对应，执行时遵循协作流程的规则。**

核心检查项：
1. **组件库能力边界检查（强制）**：检查 `node_modules/@douyinfe/semi-ui/` 实际安装版本和导出清单
2. **选布局**：从 `data/layouts.json` 匹配候选布局
3. **选组件**：从 `data/components-schema.json` 查询组件
4. **呈现方案** → **用户确认** → **应用 Token**

> 若用户直接提供截图/设计稿，可跳过 Round 1~2 确认环节，但组件边界检查不可跳过。

### Step 3: 确定项目模式

生成前必须先确认项目模式，避免重复创建 node_modules：

**检测逻辑：**
1. 检查当前工作目录是否已有 `package.json` + `vite.config.js` → 说明已有 Vite 项目
2. 如果有，**询问用户**："检测到当前目录已有 Vite 项目，是否复用现有项目？（复用 / 新建独立项目）"
3. 如果没有，**询问用户**："生成单页面原型还是多页面原型项目？"

**模式 A：单页面独立项目（默认）**
- 适用场景：只生成一个页面，独立交付
- 目录结构：
```
{outputDir}/
├── package.json
├── vite.config.js
├── index.html
└── src/
    ├── main.jsx
    ├── App.jsx           # 单个页面组件
    └── index.css
```

**模式 B：多页面原型项目（批量生成）**
- 适用场景：需要连续生成多个页面原型，共享组件和依赖
- 目录结构：
```
{projectDir}/
├── package.json          # 共享依赖，只安装一次
├── vite.config.js        # 配置多入口或 React Router
├── index.html
└── src/
    ├── main.jsx
    ├── App.jsx           # 路由根组件或页面导航
    ├── index.css
    ├── pages/            # 页面组件目录
    │   ├── Page1.jsx
    │   ├── Page2.jsx
    │   └── ...
    └── components/       # 共享组件（可选）
        └── SharedNav.jsx
```

**复用已有项目：**
- 只生成新的页面组件到 `src/pages/`（或用户指定目录）
- 不覆盖 `package.json`、`vite.config.js`、`main.jsx`
- 如果页面需要路由，提示用户在 `App.jsx` 中添加路由配置

> **建议**：如果用户说"生成一套系统"、"多个页面"、"批量"等关键词，优先推荐模式 B；如果用户只给了一个具体页面需求，用模式 A。

### Step 4: 生成项目文件

> **生成方案：Vite 工程化（实际工程项目标准，组件还原度 100%）**

**单页面模式目录结构：**
```
{outputDir}/
├── package.json          # 依赖声明（react, react-dom, @douyinfe/semi-ui, @douyinfe/semi-icons）
├── vite.config.js        # Vite 配置 + @vitejs/plugin-react
├── index.html            # HTML 入口
└── src/
    ├── main.jsx          # React 根挂载 + Semi CSS 引入
    ├── App.jsx           # 主页面组件（从 layouts 和 components 组装）
    └── index.css         # Token CSS 变量 + 全局样式
```

**多页面模式目录结构：**
```
{projectDir}/
├── package.json
├── vite.config.js
├── index.html
└── src/
    ├── main.jsx
    ├── App.jsx           # 含路由配置或页面导航
    ├── index.css
    ├── pages/
    │   └── {PageName}.jsx    # 本次生成的页面
    └── components/
        └── SharedNav.jsx     # 共享导航（如需要）
```

**package.json 模板：**
```json
{
  "name": "ue-skill-page",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@douyinfe/semi-ui": "^2.27.0",
    "@douyinfe/semi-icons": "latest",
    "@douyinfe/semi-illustrations": "latest"
  },
  "devDependencies": {
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.0.8"
  }
}
```

**vite.config.js 模板：**
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    host: true
  }
})
```

**index.html 模板：**
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{页面标题}</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
</html>
```

**src/main.jsx 模板：**
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import '@douyinfe/semi-ui/dist/css/semi.css'
import '@douyinfe/semi-icons/dist/css/semi-icons.css'
import './index.css'
import App from './App.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

**src/index.css 模板：**
```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  /* 从 tokens.json 注入的 CSS 变量 */
  --color-primary: #2993E8;
  --color-bg-primary: #FFFFFF;
  --color-bg-secondary: #F7F8FA;
  --color-text-primary: #1D2129;
  --color-text-secondary: #4E5969;
  --color-border-subtle: #E5E6EB;
  --color-border: #C9CDD4;
  --color-success: #00B42A;
  --color-warning: #FF7D00;
  --color-error: #F53F3F;
  --font-size-body: 14px;
  --font-size-h3: 16px;
  --font-size-h2: 18px;
  --font-size-h1: 24px;
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --radius-default: 6px;
  --radius-large: 8px;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background-color: var(--color-bg-secondary);
  color: var(--color-text-primary);
  margin: 0;
  -webkit-font-smoothing: antialiased;
}

/* 0.5px 边框工具类 */
.border-05 {
  border: 0.5px solid var(--color-border-subtle);
}

/* 全局移除 box-shadow 偏好 */
.no-shadow {
  box-shadow: none !important;
}
```

**组件写法（App.jsx）：**
- 使用标准 JSX 语法，`.jsx` 文件
- 组件从 `@douyinfe/semi-ui` 直接 import：`import { Button, Input } from '@douyinfe/semi-ui'`
- 图标从 `@douyinfe/semi-icons` import：`import { IconHome } from '@douyinfe/semi-icons'`
- 使用 `ReactDOM.createRoot` 挂载（React 18 标准）
- 严格使用 Semi 组件，禁止 HTML+CSS 冒充（见硬门禁）
- **子组件写法与官方文档 1:1 对齐**：`Radio.Group`、`Collapse.Panel`、`Tabs.TabPane`、`Nav.Item`、`Form.Input` 等

### Step 5: 安装依赖并启动预览

生成项目文件后，执行：

```bash
cd {outputDir}
npm install
npm run dev
```

Vite 开发服务器启动后，在浏览器中打开 `http://localhost:5173` 预览页面。

## 硬门禁

| 规则 | 说明 |
|------|------|
| **必须使用 Semi 组件** | 所有可交互元素（输入框、按钮、选择器、列表、头像、标签、弹窗等）必须使用 `@douyinfe/semi-ui` 中的真实组件，以标准 JSX import 方式编写 |
| **严禁 HTML+CSS 冒充组件** | 禁止用 `<button>`、`<input>`、`<textarea>`、`<select>` 等原生标签替代 Semi Button/Input/TextArea/Select；禁止手写 CSS 模拟组件外观 |
| **子组件与官方文档 1:1** | 必须按官方文档写法使用子组件：`Radio.Group`、`Collapse.Panel`、`Tabs.TabPane`、`Nav.Item`、`Breadcrumb.Item` 等 |
| **组件边界检查不可跳过** | 动手写代码前，必须检查 `node_modules/@douyinfe/semi-ui/lib/es/index.js` 导出清单，确认是否有专用组件（如 AIChatDialogue、AIChatInput）。禁止凭经验判断"没有组件" |
| **唯一例外** | 仅当所需组件不在 `@douyinfe/semi-ui` 主包中（如 VChart）且无等效替代时，才允许 HTML fallback，但必须在代码中标注原因。AI 系列组件（AIChatDialogue、AIChatInput 等）若已导出，必须使用 |
| **AI 对话场景默认完整方案** | 当需求涉及 AI 对话时，禁止默认只给最简单的 `AIChatDialogue` + `AIChatInput`。必须主动提出包含 `Reasoning`（思考块）、`Step`（执行步骤）、`Annotation`（引用标注）的完整方案，由用户确认后实施 |
| **AI 输入框默认配置区域** | 当需求涉及 AI 对话时，`AIChatInput` 必须默认配置 `renderConfigureArea`，包含至少三项：`Configure.Button`（联网搜索开关）、`Configure.Select`（知识库选择）、`Configure.RadioButton`（快速/思考模式切换）。用户明确拒绝时才可省略 |
| **图标名强制验证** | 生成含图标的代码前，**必须先 `Read data/icons.json`**，在 523 个图标清单中确认每个 import 的图标名存在。**禁止凭语义猜测图标名**（如想要"建筑"就写 IconBuilding、想要"树"就写 IconTree——这些都不存在）。附录 D 仅为常用速查，权威来源是 `data/icons.json` |

## UMD vs ESM 组件写法对照（必读）

Vite 工程使用 ESM 构建，组件挂载关系完整还原官方文档。以下是与旧 CDN 方案的关键差异：

| 组件关系 | CDN/UMD（旧方案，错误） | Vite/ESM（正确，官方写法） |
|----------|------------------------|---------------------------|
| Radio 分组 | `<RadioGroup>` | `<Radio.Group>` |
| Collapse 面板 | `<Panel>` | `<Collapse.Panel>` |
| Tabs 面板 | `const TabPane = Tabs.TabPane` | `<Tabs.TabPane>` |
| Nav 子项 | `<NavItem>` | `<Nav.Item>` |
| Breadcrumb 子项 | `<BreadcrumbItem>` | `<Breadcrumb.Item>` |
| Pagination 当前页 | `currentPage={page}` | `currentPage={page}` |
| Pagination 回调 | `onPageChange={fn}` | `onPageChange={fn}` |
| Fragment | `<React.Fragment>` | `<>` 或 `<React.Fragment>` |

## 设计约束

| 规则 | 做法 |
|------|------|
| 边框 | 0.5px solid var(--color-border-subtle)，无 box-shadow |
| 装饰 | 无色条、无图标装饰、无 ::before/::after 伪元素 |
| 背景 | 纯色 var(--color-bg-primary/bg-secondary)，无渐变 |
| 色值 | 必须用 var(--color-*) 变量，禁止硬编码 |
| 字号 | 用 --font-size-body(14px) / --font-size-h3(16px) 等已定义值 |
| 间距 | 用 --spacing-* 系统（4px 基础） |
| 圆角 | 用 --radius-default(6px) / --radius-large(8px) 等已定义值 |

## 交互设计模式

### 可点击卡片（Clickable Card）

**适用场景**：卡片承载实体对象（企业、用户、文档等），点击后进入详情页。

**核心原则**：
1. **hover 区域必须等于点击区域** —— 整卡片可点，hover 效果覆盖整卡片
2. **无嵌套链接** —— 卡片内部不再放置任何 `<Link>` 或 `<a>` 元素
3. **视觉反馈** —— border 变色 + 微阴影，cursor 变为 pointer

**实现步骤：**

1. **用 `<Link>` 包裹 `<Card>`**（或外层 `<div>` 包 `<Card>` + `onClick` 跳转）
2. **内部文本用 `<Text>` 代替 `<Link>`** —— 避免嵌套链接警告
3. **hover 状态管理**：
   ```jsx
   const [hoveredCard, setHoveredCard] = useState(null)
   ```
4. **Card 样式**：
   ```jsx
   <Card
     style={{
       border: hoveredCard === item.key
         ? '0.5px solid var(--semi-color-primary)'
         : '0.5px solid var(--color-border-subtle)',
       boxShadow: hoveredCard === item.key ? '0 2px 8px rgba(0,0,0,0.06)' : 'none',
       transition: 'border-color 0.2s, box-shadow 0.2s',
       cursor: 'pointer',
     }}
     onMouseEnter={() => setHoveredCard(item.key)}
     onMouseLeave={() => setHoveredCard(null)}
   >
   ```

**完整示例：**
```jsx
import { Card, Text } from '@douyinfe/semi-ui'
import { Link } from 'react-router-dom'

const ClickableCard = ({ item }) => {
  const [hovered, setHovered] = useState(false)

  return (
    <Link to={`/detail/${item.id}`} style={{ textDecoration: 'none', display: 'block' }}>
      <Card
        style={{
          border: hovered
            ? '0.5px solid var(--semi-color-primary)'
            : '0.5px solid var(--color-border-subtle)',
          boxShadow: hovered ? '0 2px 8px rgba(0,0,0,0.06)' : 'none',
          transition: 'border-color 0.2s, box-shadow 0.2s',
          cursor: 'pointer',
        }}
        bodyStyle={{ padding: '14px 16px' }}
        onMouseEnter={() => setHovered(true)}
        onMouseLeave={() => setHovered(false)}
      >
        <Text style={{ fontSize: 16, fontWeight: 600 }}>{item.name}</Text>
        {/* 其他内容 */}
      </Card>
    </Link>
  )
}
```

**参数速查：**

| 属性 | 默认值 | 说明 |
|------|--------|------|
| 边框宽度 | 0.5px | 符合设计约束 |
| hover 边框色 | `var(--semi-color-primary)` | 主题蓝 |
| 默认边框色 | `var(--color-border-subtle)` | 浅灰 |
| hover 阴影 | `0 2px 8px rgba(0,0,0,0.06)` | 极轻微，不喧宾夺主 |
| 过渡时间 | 0.2s | 快速响应 |
| cursor | pointer | 明确可点击暗示 |

**禁忌：**
- ❌ 不要在 Card 内部再放 `<Link>` 或 `<a>` —— 导致嵌套链接，React 报错
- ❌ hover 效果不要只做在标题上 —— 用户会误以为只有标题可点
- ❌ 不要用 `onClick` 包裹 Card 后内部再加链接 —— 事件冒泡混乱

**无障碍补充：**
- 添加 `role="link"` 和 `tabIndex={0}` 到外层包裹元素（如未使用 `<Link>`）
- 支持 Enter/Space 键触发跳转（键盘导航）

## 组件与布局速查

组件映射和布局模板已外置到数据文件，**禁止在本 SKILL 正文中维护映射表**：

- 查组件映射 → 读取 `data/components-schema.json`
- 查布局模板 → 读取 `data/layouts.json`

如需快速了解有哪些组件/布局可用，通过 `Read` 工具读取对应文件后查看。

---




## 附录 D：常用图标速查（@douyinfe/semi-icons）

> **⚠️ 权威来源为 `data/icons.json`（523 个）。本附录仅为**常用图标**快速参考。生成代码前必须验证。**

### 对象与场所
| 图标名 | 用途 | 常见错误名 |
|--------|------|-----------|
| `IconHome` / `IconHomeStroked` | 首页/校园 | |
| `IconApartment` | 建筑/区域/楼层 | `IconBuilding` ❌ |
| `IconComponent` / `IconComponentStroked` | 组件/模块/子节点 | `IconTree` ❌、`IconModule` ❌ |
| `IconMapPin` / `IconMapPinStroked` | 位置/点位 | |
| `IconMapPinStroked` | 位置（线框） | |
| `IconGlobe` / `IconGlobeStroked` | 全局/国际 | |
| `IconDesktop` | 桌面/设备 | |
| `IconServer` / `IconServerStroked` | 服务器 | |
| `IconWifi` | 网络/WiFi | |
| `IconSignal` | 信号 | |
| `IconSmartPhoneStroked` | 手机 | |
| `IconTabletStroked` | 平板 | |
| `IconMonitorStroked` | 显示器 | |
| `IconKey` | 密钥/钥匙 | |
| `IconShield` / `IconShieldStroked` | 安全/防护 | |
| `IconTag` / `IconTagStroked` | 标签 | |
| `IconPriceTag` / `IconPriceTagStroked` | 价格标签 | |
| `IconGift` / `IconGiftStroked` | 礼物 | |
| `IconTrophy` / `IconTrophyStroked` | 奖杯 | |
| `IconRocket` / `IconRocketStroked` | 火箭/快速 | |
| `IconLightning` / `IconLightningStroked` | 闪电/快速操作 | |
| `IconThumbsUp` / `IconThumbsUpStroked` | 点赞 | |
| `IconThumbsDownStroked` | 踩 | |
| `IconLikeThumb` | 喜欢 | |
| `IconTriangleUp` / `IconTriangleDown` | 三角箭头 | |
| `IconTriangleLeft` / `IconTriangleRight` | 三角左右 | |
| `IconTreeTriangleDown` | 树展开 | `IconTree` ❌ |
| `IconTreeTriangleRight` | 树收起 | `IconTree` ❌ |
| `IconBranch` | 分支 | |
| `IconPulse` | 脉冲/活动 | |
| `IconActivity` | 活动 | |
| `IconLayer` | 图层 | |
| `IconCube` | 立方体 | |
| `IconColorPalette` | 调色板 | |
| `IconCamera` | 相机 | |
| `IconVideo` / `IconVideoStroked` | 视频 | |
| `IconImage` / `IconImageStroked` | 图片 | |
| `IconMusic` / `IconMusicNoteStroked` | 音乐 | |
| `IconPlay` / `IconPlayStroked` | 播放 | |
| `IconPause` / `IconPauseStroked` | 暂停 | |
| `IconStop` / `IconStopStroked` | 停止 | |
| `IconVolume` / `IconVolumeMute` / `IconVolumeStroked` | 音量 | |

### 导航与操作
| 图标名 | 用途 | 常见错误名 |
|--------|------|-----------|
| `IconHome` | 首页 | |
| `IconSearch` | 搜索 | |
| `IconSend` / `IconSendStroked` | 发送 | |
| `IconClose` | 关闭/取消 | |
| `IconDelete` / `IconDeleteStroked` | 删除 | |
| `IconPlus` / `IconPlusCircle` | 添加 | |
| `IconMinus` / `IconMinusCircle` | 减少/移除 | |
| `IconEdit` / `IconEditStroked` | 编辑 | |
| `IconSetting` | 设置 | |
| `IconMore` / `IconMoreStroked` | 更多 | |
| `IconMenu` | 菜单 | |
| `IconLink` | 链接 | `IconChain` |
| `IconUnlink` | 取消链接 | |
| `IconExport` | 导出 | |
| `IconImport` | 导入 | |
| `IconSave` / `IconSaveStroked` | 保存 | |
| `IconPrint` | 打印 | |
| `IconShare` / `IconShareStroked` | 分享 | |
| `IconDownload` | 下载 | |
| `IconUpload` | 上传 | |
| `IconRefresh` / `IconSync` | 刷新/同步 | |
| `IconRedo` / `IconUndo` | 重做/撤销 | |
| `IconCopy` / `IconCopyStroked` | 复制 | |
| `IconLock` / `IconUnlock` | 锁/解锁 | |

### 状态与反馈
| 图标名 | 用途 | 常见错误名 |
|--------|------|-----------|
| `IconCheckboxTick` | 勾选/通过 | `IconCheck` ❌ |
| `IconCheckList` | 清单/审批 | `IconApproval` ❌ |
| `IconClose` | 关闭/拒绝 | |
| `IconCrossStroked` | 叉号 | `IconCross` ❌ |
| `IconCrossCircleStroked` | 圆形叉号 | |
| `IconCheckChoiceStroked` | 勾选（选择） | |
| `IconCheckCircleStroked` | 圆形勾选 | |
| `IconAlertCircle` | 警告圆形 | |
| `IconAlertTriangle` | 警告三角 | |
| `IconInfoCircle` | 信息 | |
| `IconHelpCircle` | 帮助 | |
| `IconStar` / `IconStarStroked` | 收藏/星标 | |
| `IconHeartStroked` / `IconLikeHeart` | 喜欢 | |
| `IconFavoriteList` | 收藏列表 | |
| `IconBookmark` | 书签 | |
| `IconFlag` / `IconFlagStroked` | 标记/旗帜 | |
| `IconPrize` / `IconPrizeStroked` | 奖励 | |
| `IconBell` / `IconBellStroked` | 通知 | |
| `IconClock` / `IconClockStroked` | 时间/历史 | `IconHistory` ❌ |
| `IconCalendar` | 日历 | |
| `IconEyeOpened` | 可见/查看 | |
| `IconEyeClosed` | 隐藏 | |

### 列表与网格
| 图标名 | 用途 | 常见错误名 |
|--------|------|-----------|
| `IconList` | 列表 | |
| `IconListView` | 列表视图 | |
| `IconOrderedList` | 有序列表 | |
| `IconGridRectangle` | 网格矩形 | |
| `IconGridSquare` | 网格方形 | |
| `IconGridView` | 网格视图 | |
| `IconTemplate` | 模板/项目 | `IconProject` ❌ |
| `IconChecklistStroked` | 任务清单 | |

### 排序与筛选
| 图标名 | 用途 |
|--------|------|
| `IconSort` / `IconSortStroked` | 排序 |
| `IconFilter` / `IconFilterStroked` | 筛选 |
| `IconAscend` | 升序 |
| `IconClear` | 清除 |
| `IconSearch` / `IconSearchStroked` | 搜索 |

### 文件与文件夹
| 图标名 | 用途 |
|--------|------|
| `IconFile` | 文件 |
| `IconFolder` / `IconFolderOpen` | 文件夹 |
| `IconArticle` | 文档/文章 |
| `IconImage` / `IconImageStroked` | 图片 |
| `IconVideo` / `IconVideoStroked` | 视频 |
| `IconMusic` / `IconMusicNoteStroked` | 音乐 |
| `IconCode` / `IconCodeStroked` | 代码 |
| `IconArchive` | 归档 |

### 用户与通讯
| 图标名 | 用途 | 常见错误名 |
|--------|------|-----------|
| `IconUser` | 用户 | |
| `IconUserGroup` | 用户组 | |
| `IconUserCircle` / `IconUserCircleStroked` | 用户头像 | |
| `IconUserAdd` | 添加用户 | |
| `IconUserSetting` | 用户设置 | |
| `IconComment` / `IconCommentStroked` | 评论/消息 | `IconChat` ❌ |
| `IconMail` / `IconMailStroked` | 邮件 | |
| `IconPhone` / `IconPhoneStroked` | 电话 | |
| `IconShareStroked` | 分享 | |
| `IconMapPin` | 位置 | |

---

## 附录 E：Semi v2.99 API 注意事项

> **规则**：以下 API 签名是 Semi UI 2.99.2 的实际行为。禁止凭其他 UI 库经验假设签名。

### Form
```js
// ✅ Form.Input 隐藏标签：用 label=""
<Form.Input field="username" label="" placeholder="用户名" />

// ❌ noLabel 属性在 2.99.2 无效，label 仍会显示 field 名
<Form.Input field="username" noLabel />

// ✅ horizontal 布局带标签
<Form layout="horizontal" labelPosition="left" labelCol={{ span: 6 }} wrapperCol={{ span: 18 }}>
```

### Toast
```js
// ✅ 正确：接受对象参数
Toast.success({ content: '操作成功' })
Toast.warning({ content: '警告信息' })
Toast.error({ content: '错误信息' })

// ❌ 错误：不接受纯字符串（这是 Ant Design 的 API）
Toast.success('操作成功')
```

### Badge
```js
// ✅ type 属性控制颜色
<Badge count={5} type="danger" />   // 红色
<Badge count={3} type="success" />  // 绿色
<Badge count={0} type="warning" />  // 黄色

// 动态角标
<Badge count={5} overflowCount={99} />
```

### Table
```js
// ✅ Column 是 Table 的静态子组件
const { Column } = Table
// 或直接 <Table.Column ... />

// ✅ Pagination
<Table pagination={{ pageSize: 10 }} />

// ✅ dataSource 需要 key 字段
dataSource={rows.map((r,i) => ({ ...r, key: i }))}
```

### Modal
```js
// ✅ 确认按钮文案
<Modal okText="确认保存" cancelText="取消" onOk={handleOk} onCancel={handleCancel} />

// ✅ 危险操作
<Modal okType="danger" okText="确认删除" />
```

### Semi CSS 变量 vs 自定义变量
```css
/* ✅ 优先使用 Semi 官方变量 */
var(--semi-color-primary)
var(--semi-color-border)
var(--semi-color-text-0)

/* ✅ 页面级布局可用自定义变量（var(--color-*)） */
/* ❌ 不可用自定义变量覆盖 Semi 组件内部样式 */
```

### 图标导入规则（关键！）
```js
// ✅ 生成代码前，必须先 Read data/icons.json 验证每个图标名
// ✅ 附录 D 仅为常用速查，完整清单见 data/icons.json（523 个）
// ❌ 禁止语义推断："建筑"没有 IconBuilding，是 IconApartment
// ❌ 禁止语义推断："树"没有 IconTree，是 IconTreeTriangleDown/Right
// ❌ 禁止语义推断："审批"没有 IconApproval，是 IconCheckList
// ❌ 禁止语义推断："对勾"没有 IconCheck，是 IconCheckboxTick
// ❌ 禁止跨库记忆

import { IconHome, IconSearch, IconSend } from '@douyinfe/semi-icons'
```
