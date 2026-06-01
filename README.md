# UE Page Builder

> 基于 **Semi Design + Vite + React** 的企业级页面生成器。数据文件外置、图标名强制验证、组件边界检查，硬门禁保障输出质量。

[![Semi UI](https://img.shields.io/badge/Semi_UI-2.99.2-2993E8)](https://semi.design/)
[![React](https://img.shields.io/badge/React-18.2-61DAFB)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-5.x-646CFF)](https://vitejs.dev/)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![Version](https://img.shields.io/badge/version-2.0.0-blue)]()

---

## 项目简介

**UE Page Builder** 是一个面向 AI Agent 的"前端页面生成技能包"，专为 WorkBuddy / Claude / GPT 等具备工具调用能力的智能体设计。

输入一句话需求（例如"做一个设备管理页"），即可输出：

- 可运行的 **Vite + React** 工程
- 基于 **Semi Design 2.99.2** 真实组件（HTML 冒充零容忍）
- **数据外置架构**：Token、组件 Schema、布局模板、图标清单全部独立维护，杜绝 AI 凭记忆幻觉

### 适合谁用

| 角色 | 价值 |
|------|------|
| **产品经理** | 把这个 SKILL 加载到 Agent，Agent 就能高质量生成企业级页面 |
| **前端工程师** | 把它作为"需求 → 代码"的脚手架，按工程目录直接 npm run dev |
| **技术管理者** | 团队的"半人马"协作标准——AI 出 90%，人 review 10% |

---

---

## 协作工作流

### 路径 A：需求明确 → 快速生成

适用：用户提供了完整页面描述、截图、设计稿。

1. **组件库能力边界检查**（不可跳过）
2. 读取三层数据文件 → 匹配布局和组件 → 生成项目 → 启动预览
3. 用户验收 → 迭代调整

### 路径 B：需求模糊 → 方案确认（推荐）

适用：用户只有一句话需求，如"做一个产业详情页"。

```
Round 0  组件库边界检查（强制）
  ↓
Round 0.5  AI 场景默认推荐完整方案
  ↓
Round 1  需求澄清 + 布局推荐
  ↓
Round 2  组件清单确认（可选）
  ↓
Round 3  生成代码
```

**核心原则**：宁可多一轮确认，也不要生成后整页推翻重来。

---



## 核心特性

### 1. 数据文件外置架构（防止幻觉）

SKILL 把所有"机器易错"的数据**从正文中抽离**，单独存为 JSON 文件：

```
SKILL.md
├── data/tokens.json         # CSS 变量定义
├── data/components-schema.json  # 组件 API 注册表
├── data/layouts.json        # 布局模板
└── data/icons.json          # 523 个已验证图标名
```

**强制规则**：生成代码前必须先 `Read` 对应 JSON 文件，**禁止凭记忆写**。这彻底解决了"组件名拼错"、"图标名瞎编"、"Token 命名混乱"三大典型幻觉问题。

### 2. 523 个图标名强制验证

`data/icons.json` 是从 `node_modules/@douyinfe/semi-icons` 实际解析出来的图标清单，包含 **523 个真实存在的图标名**。】

### 3. AI 对话场景完整方案

当用户需求涉及"AI 对话"、"智能助手"、"类 ChatGPT"等关键词时，**自动默认推荐包含过程可视化的完整方案**：

| 用户关键词 | 默认推荐子组件 | 效果 |
|-----------|---------------|------|
| 思考过程 / 推理链 / 为什么 | `AIChatDialogue.Reasoning` | 可折叠的思考过程 |
| 分步执行 / 任务进度 | `AIChatDialogue.Step` | 搜索/读取/生成的步骤展示 |
| 引用来源 / 参考材料 | `AIChatDialogue.Annotation` | 链接/标题/描述引用 |
| 联网搜索 / 知识库 | `AIChatInput.Configure` | 输入框配置区 |

### 4. 组件库能力边界检查

动手写代码前，**必须先检查 `node_modules/@douyinfe/semi-ui/` 实际安装版本和导出清单**，确认是否有专用组件可用。

---

## 技术栈

| 类别 | 选型 | 版本 |
|------|------|------|
| 框架 | React | 18.2.0 |
| 构建 | Vite | 5.0.8+ |
| 组件库 | @douyinfe/semi-ui | ^2.27.0（实测 2.99.2） |
| 图标 | @douyinfe/semi-icons | latest |
| 插画 | @douyinfe/semi-illustrations | latest |
| 插件 | @vitejs/plugin-react | ^4.2.1 |

---

## 项目结构

```
ue-page-builder/
├── SKILL.md              # 主技能文件（28.7KB）
├── _meta.json            # 元数据（slug/version/author）
├── README.md             # 本文件
└── data/
    ├── tokens.json            # CSS Token 定义
    ├── components-schema.json # 组件 API Schema
    ├── layouts.json           # 布局模板
    └── icons.json             # 523 个已验证图标
```

## 硬门禁规则（8 条强制规则）

| # | 规则 | 违反后果 |
|---|------|---------|
| 1 | **必须使用 Semi 组件** | 所有可交互元素必须 `@douyinfe/semi-ui` import，禁原生 HTML 标签 |
| 2 | **严禁 HTML+CSS 冒充组件** | 禁止 `<button>` 替代 Button、`<input>` 替代 Input |
| 3 | **子组件与官方文档 1:1** | `<Radio.Group>` 而非 `<RadioGroup>`、`<Tabs.TabPane>` 而非 `<TabPane>` |
| 4 | **组件边界检查不可跳过** | 必须看 `node_modules/@douyinfe/semi-ui/lib/es/index.js` 导出清单 |
| 5 | **唯一例外** | 仅当组件不在主包中（如 VChart）才允许 HTML fallback，且必须标注原因 |
| 6 | **AI 对话场景默认完整方案** | 禁止默认只给最简单的 `AIChatDialogue` 气泡，必须含 Reasoning/Step/Annotation |
| 7 | **AI 输入框默认配置区域** | `renderConfigureArea` 必须含 `Button/Select/RadioButton` 至少 3 项 |
| 8 | **图标名强制验证** | 必先 `Read data/icons.json`，523 个图标清单，禁凭语义猜测 |

---

## 数据文件说明

### `data/tokens.json` — 设计 Token（15.6KB）

定义所有 CSS 变量：颜色（primary/bg/text/border/state）、字号、间距、圆角、阴影、动画时长。

### `data/components-schema.json` — 组件 API（51.4KB）

Semi UI 组件的注册表：组件名、import 路径、必需 props、常用 props、典型用法。生成代码时**先查这里再写**。

### `data/layouts.json` — 布局模板（2.3KB）

预定义的页面骨架（基础布局 baseLayouts + 业务页面 pageLayouts），如"管理后台 + 侧边栏"、"列表页 + 筛选条"、"详情页 + Tabs"。

### `data/icons.json` — 图标清单（11.9KB）

**523 个真实存在的 semi-icons 图标名**。从 `@douyinfe/semi-icons/lib/es/icons/index.js` 实际解析得到。这是图标名的**权威来源**。

---

## 使用方式

### 1. 解压到 skills 目录

```bash
# 复制 SKILL 包到用户级 skills 目录
unzip ue-page-builder-v2.0.0.zip -d ~/.workbuddy/skills/

# 目录结构应为
~/.workbuddy/skills/ue-page-builder/
├── SKILL.md
├── _meta.json
└── data/
```

### 2. 在 WorkBuddy / Agent 中调用

```
加载技能：ue-page-builder
输入需求："做一个设备管理页"
```

### 3. 作为脚手架使用

```bash
# 1. 用 SKILL 生成 src/App.jsx
# 2. 创建 Vite 项目
npx create-vite my-page --template react
cd my-page

# 3. 复制生成的代码到 src/App.jsx
# 4. 安装依赖
npm install @douyinfe/semi-ui @douyinfe/semi-icons @douyinfe/semi-illustrations

# 5. 启动
npm run dev
# 浏览器打开 http://localhost:5173
```

---

## 关键示例

### Vite 工程入口

```jsx
// src/main.jsx
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

### Token 注入

```css
/* src/index.css */
:root {
  --color-primary: #2993E8;
  --color-bg-primary: #FFFFFF;
  --color-bg-secondary: #F7F8FA;
  --color-text-primary: #1D2129;
  --color-text-secondary: #4E5969;
  --color-border-subtle: #E5E6EB;
  --spacing-md: 16px;
  --radius-default: 6px;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background-color: var(--color-bg-secondary);
  color: var(--color-text-primary);
  margin: 0;
}
```


## 版本历史

### v2.0.0（2025-05-31）

- 🆕 新增 `data/icons.json`（523 个真实图标清单）
- 🆕 新增"AI 对话场景默认完整方案"规则
- 🆕 新增"AI 输入框默认配置区域"规则
- 🆕 新增"图标名强制验证"硬门禁
- ♻️  数据文件全部外置（tokens / components-schema / layouts）
- ♻️  删除 SKILL 正文中重复的组件映射表
- 📝  附录 D 扩展为 50+ 常用图标速查

### v1.x

- 初始版本，内联所有数据

---

## 路线图

- [ ] 接入 Tailwind / CSS-in-JS 的可选 token 适配层
- [ ] 自动从 `node_modules` 提取 `components-schema.json` 的脚本
- [ ] 支持 Ant Design / Element Plus 版本（用户呼声）
- [ ] AI 聊天子组件可视化编辑器

---

## 贡献指南

1. Fork 本仓库
2. 创建 feature 分支：`git checkout -b feature/xxx`
3. 更新 `data/` 中的 JSON 时，请附带：a) 真实来源（如 `node_modules/...`）b) 验证截图/输出
4. 修改 `SKILL.md` 时遵守：a) 不在正文中维护数据表 b) 引用强制 `Read` 指令
5. 提交 PR 并描述变更动机

---

## 许可

[MIT](./LICENSE) © 2025 WorkBuddy

---

## 致谢

- [Semi Design](https://semi.design/) — 字节跳动出品的企业级 UI 库
- [Vite](https://vitejs.dev/) — 下一代前端构建工具
- 所有用这个 SKILL 生成过页面的工程师和 PM
