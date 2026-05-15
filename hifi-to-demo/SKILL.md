---
name: hifi-to-demo
description: |
  将高保真设计稿转换为可运行代码，分两阶段交付：
  第一阶段生成单文件快速 Demo 供设计评审与迭代；
  确认设计方向后，第二阶段输出工程级代码，可直接介入真实开发仓库。
  当用户提供 UI 设计（截图、Figma 链接或详细描述）并要求「转成代码」「做前端 Demo」
  「生成可运行原型」「工程化」「接入项目」时触发此 skill。
  中文触发词：帮我把这个设计转成代码 / 做一个前端demo / 生成可运行的Demo /
  把设计稿变成代码 / 做个可以点击的原型 / 工程化这个Demo / 转成组件 / 接入现有项目。
  English triggers: "turn this into code", "make a demo", "generate a frontend prototype",
  "engineering-ready code", "production components".
  只要同时存在「设计稿」和「写代码」的意图，无论用户是否明确说 demo，都应调用此 skill。
---

# 高保真设计稿 → Demo → 工程代码 两阶段工作流

本 skill 提炼自 openUBMC Studio L1 协作实践。
核心理念：**Demo 是验证工具，不是最终交付物。**
设计意图通过 Demo 快速对齐之后，工程代码才是真正能进仓库的资产。

---

## 需求分析

### 用户

| 角色 | 使用场景 | 关注点 |
|------|---------|--------|
| 体验设计师 | 将设计稿转化为可演示的原型或工程代码 | 设计意图能被准确还原；交互细节不被遗漏 |
| 前端开发工程师 | 接收设计侧的工程代码，直接集成进项目 | 代码规范一致；类型完整；能直接跑起来 |
| 产品经理 / 利益相关者 | 评审 Demo 原型，确认设计方向 | 能在浏览器里直接看，不需要安装任何东西 |
| 交互设计师 | 审阅交互细化文档，补充确认清单 | 状态枚举完整；边缘情况有明确提案 |

### 场景

1. **快速原型验证**：设计稿刚完成，需要在利益相关者评审前做可点击原型——Demo 阶段，单文件，零安装。
2. **设计交付开发**：设计方向已对齐，开发需要能直接集成的组件代码——工程阶段，类型完整、状态覆盖。
3. **多轮迭代**：Demo 评审后有修改意见，在 Phase 1 内快速迭代，方向稳定后再生成工程代码，避免在工程代码上来回改。
4. **设计资产沉淀**：不只是代码，还有结构文档、交互说明、质量报告，形成可复用的协作资产。

### 解决的问题

- **设计-开发鸿沟**：设计稿是静态图，开发理解偏差导致实现走样。通过 `design-structure.md` 将设计意图结构化，消除歧义。
- **交互细节丢失**：高保真图无法表达状态枚举和边缘情况。`interaction-spec.md` 的「待确认清单」把隐式假设显式化，在实现前对齐。
- **Demo 代码进工程**：Demo 代码将就、无类型、Mock 数据硬编码，直接进工程会留下技术债。两阶段明确分工，各自达到各自的质量标准。
- **方向未定就写工程代码**：在设计还在迭代时就写工程级代码，一旦方向变了返工成本极高。Phase 1 → 确认节点 → Phase 2 的流程从结构上避免了这个问题。

### 设计点

- **两阶段分离**：Demo 优化「能不能看」，工程代码优化「能不能用」，目标不同、标准不同，不能合并成一次交付。
- **Mock 数据即 API 契约**：Demo 的字段命名从 Phase 1 就对齐后端约定，Phase 2 的 API 层是替换而非重写。
- **确认节点是真正的关口**：Phase 1 完成后的确认不是礼节，是最便宜的变更窗口——Demo 层改比工程层改快得多。
- **文档是协作界面**：README / CHANGELOG / BRANCH / INTEGRATION 不是形式主义，是设计侧产出与开发侧接收之间的契约文件。

---

## 两阶段交付结构

### Phase 1 · Demo（设计验证）

| # | 文件 | 用途 |
|---|------|------|
| 1 | `design-structure.md` | 布局层级、组件清单、色彩 Token、数据接口 |
| 2 | `interaction-spec.md` | 可交互区域、状态枚举、交互序列、**待设计师确认清单** |
| 3 | `tech-selection.md` | 技术选型（Demo 与工程各一套）、迁移路径 |
| 4 | `demo/index.html` | 含 Mock 数据的单文件可运行原型，双击即开 |
| 5 | `quality-review.md` | 易用性审查，标注已修复 / 已知 / 需设计师介入的问题 |
| 6 | `prompt-asset.md` | 原始 Prompt、已知局限、下一步迭代建议 |

### Phase 2 · 工程代码（进仓库）

> 在 Phase 1 Demo 经过充分评审、设计方向确认后才执行。

| # | 文件 | 用途 |
|---|------|------|
| 7 | `src/components/[Name]/index.tsx`（或 `.vue`） | 组件主体，符合项目框架规范 |
| 8 | `src/components/[Name]/types.ts` | 完整 TypeScript 类型定义，含 Props / Events / Emits |
| 9 | `src/services/[feature]Api.ts` | API 集成层，将 Mock 字段映射到真实接口调用约定 |
| 10 | `src/components/[Name]/[Name].test.ts` | 单元测试桩，覆盖核心状态分支 |
| 11 | `INTEGRATION.md` | 接入指南：安装依赖、配置路由、替换 Mock 数据、注意事项 |
| 12 | `README.md` | 项目总览，覆盖 Phase 1 + Phase 2 |
| 13 | `CHANGELOG.md` | 版本日志，区分 Demo 版与工程版 |
| 14 | `BRANCH.md` | 分支管理方案 |

**条件产出**（仅在参考开源代码时生成）：

| # | 文件 | 用途 |
|---|------|------|
| 15 | `learning/[库名].md` | 开源代码结构拆解与学习笔记 |

---

## Phase 1 工作流 — Demo 快速验证

### 第 1 步 — 收集输入

主动询问不明确的地方：缺失的色彩 Token、边缘状态的交互行为、数据 Schema。
不要猜测会实质影响实现的内容。

同时确认：
- 是否涉及参考开源代码？若是，记录库名，后续生成 `learning/` 文档。
- 是否为团队协作项目？若是，确认分支命名规范偏好。
- **目标开发栈是什么？**（React / Vue 3 / 原生 HTML，框架版本，CSS 方案）—— 工程代码需要对齐，Phase 1 阶段提前了解。

### 第 2 步 — 分析设计 · 结构 + 元素描述

**写任何代码之前**，先完成分析并写入 `design-structure.md`：

**2a. 布局层级**
- 页面整体分区（顶栏 / 侧边栏 / 主区 / 底部等）
- 各区域尺寸与间距系统

**2b. 组件清单 + 元素详细描述**
对每个独立 UI 组件逐一描述：
- 组件名称与类型（容器 / 展示 / 输入 / 导航等）
- 视觉细节：背景色、边框、圆角、阴影、字体规格（均用 hex 值）
- 内部元素层叠关系
- 与相邻组件的间距关系
- 空状态 / 加载状态下的外观

**2c. 色彩 Token** — 提取所有 hex 值

**2d. 数据接口** — TypeScript 风格接口定义，字段命名与预期后端 API 保持一致

**2e. 非静态区域标注**
- 动态数据区 / 骨架屏区域 / 条件渲染区 / 动画区

若用户要求「先看分析」，在此暂停并等待确认。

### 第 3 步 — 交互分析 + 可行性评估

写入 `interaction-spec.md`：

**3a. 可交互区域**
对每个可交互元素：触发方式 → 系统响应 → 状态枚举 → 逆操作路径

**3b. 待设计师确认清单**
在文件末尾单独列「⚠️ 待设计师确认」区块：
动画时长与缓动曲线 / 错误提示文案与时机 / 边缘状态视觉处理 / 多步骤流程回退规则

**3c. 可行性与拓展性评估**
- Demo 阶段能否实现（完整 / Mock / 占位）
- 接入真实数据后是否需要重构
- 影响后续拓展的架构决策

### 第 4 步 — 技术选型（两套）

在 `tech-selection.md` 中分别记录：

**Demo 选型原则**：单文件、零安装、双击即开。
CDN 引入框架，Mock 数据内联，不依赖构建工具。

**工程选型原则**：对齐用户确认的目标开发栈。

| 产品类型 | 推荐工程技术栈 |
|---------|--------------|
| 图 / 拓扑可视化 | Vue 3 + ReactFlow 或 Cytoscape.js + ELK/dagre |
| 数据密集型 Dashboard | React + shadcn/ui + Recharts + TanStack Query |
| 表单密集型企业工具 | Vue 3 + Element Plus + Pinia |
| 移动端消费者应用 | React + Tailwind CSS + React Query |
| SaaS 管理后台 | React + Tailwind CSS + Zustand |
| BMC / 嵌入式管理界面 | 原生 HTML + CSS + JS（无框架，保持轻量） |

记录：Demo 到工程的迁移路径、被排除的方案及原因。

### 第 5 步 — 生成 Demo 代码

分层构建 `demo/index.html`：
1. 静态 HTML 结构（正确的语义元素）
2. 样式（颜色 / 字体 / 间距与设计稿对齐）
3. 交互（状态变更、过渡动画、事件处理器）
4. Mock 数据（字段命名与真实 API 约定一致，至少 5–8 条记录）

### 第 6 步 — 易用性审查

代码生成后先自查，写入 `quality-review.md`：

| 维度 | 检查项 |
|------|--------|
| 可读性 | 文字对比度；字号（正文 < 12px 即警告） |
| 层级清晰度 | 主次按钮视觉权重差异；标题与正文可区分 |
| 交互反馈 | hover / active / disabled 状态明确 |
| 布局稳定性 | 动态内容填入后无布局抖动；长文本有截断处理 |
| 空状态 | 列表 / 图表为空时有提示，非空白区域 |
| 错误状态 | 表单错误提示清晰、位置靠近出错字段 |

标注每个问题：✅ 已修复 / ⚠️ 已知但不阻碍核心流程 / ❌ 需设计师重新定义

---

## ⏸ Phase 1 完成后的确认节点

Demo 完成后，向用户汇报：
1. **Demo 可以在哪里打开**（文件路径）
2. **质量审查结论**（一句话）
3. **待设计师确认的问题清单**（从 `interaction-spec.md` 摘取）
4. **提示下一步**：「如果设计方向确认，可以开始生成工程代码。请告知目标仓库的框架和目录结构。」

**除非用户明确说「继续工程化」或「生成工程代码」，不要自动进入 Phase 2。**
这个节点是真正的评审关口——Demo 上暴露的问题比工程代码阶段修便宜得多。

---

## Phase 2 工作流 — 工程级代码

> 前提：Phase 1 Demo 已通过评审，用户已确认设计方向和目标开发栈。

进入 Phase 2 前，主动了解：
- 目标仓库的目录结构约定（`components/` 在哪？API 层在哪？）
- 状态管理方案（Redux / Pinia / Zustand / 无）
- CSS 方案（CSS Modules / Tailwind / styled-components / BEM）
- 测试框架（Vitest / Jest / 无）
- TypeScript 严格度（strict 模式 / 宽松 / 纯 JS）

### 第 7 步 — 组件拆分与工程结构规划

根据 Demo 和目标仓库约定，规划组件树：

```
src/
├── components/
│   └── [FeatureName]/
│       ├── index.tsx           # 组件入口
│       ├── types.ts            # Props / Events / 共享类型
│       ├── [SubComponent].tsx  # 子组件（如有）
│       └── [FeatureName].test.ts
├── services/
│   └── [feature]Api.ts         # API 集成层
└── (hooks / store / utils — 视项目结构而定)
```

在开始写代码之前，把组件树和文件结构告诉用户确认——这一步返工成本最高。

### 第 8 步 — 生成工程组件

**组件质量标准：**

- **TypeScript 严格**：Props 类型完整，无 `any`，Emits / Events 有类型声明
- **无硬编码**：颜色、文字、枚举值提取为常量或 Token，不内联在 JSX/模板里
- **状态全覆盖**：loading / error / empty / disabled 状态均有对应 UI，不遗漏
- **无副作用泄漏**：useEffect 有清理函数；Vue 组件在 onUnmounted 移除监听器
- **可访问性（a11y）**：ARIA 标签、键盘导航、焦点管理符合 WCAG 2.1 AA 基线
- **无直接 DOM 操作**：不用 `document.querySelector`，用框架的 ref / $el 机制
- **CSS 不污染全局**：CSS Modules / scoped / styled-components，类名有命名空间
- **错误边界**：关键数据加载失败时有 fallback UI，不白屏

### 第 9 步 — API 集成层

将 Demo 的 Mock 数据替换为真实 API 调用骨架，写入 `src/services/[feature]Api.ts`：

```typescript
// 保留 Mock 作为开发降级，用环境变量或 flag 切换
const USE_MOCK = import.meta.env.VITE_USE_MOCK === 'true'

export async function fetchFeatureData(params: QueryParams): Promise<FeatureData[]> {
  if (USE_MOCK) return mockFeatureData
  const res = await http.get('/api/feature', { params })
  return res.data
}
```

字段命名严格对齐 Phase 1 中 `design-structure.md` 定义的数据接口——这是 Mock 数据即 API 契约的落地。

### 第 10 步 — 单元测试桩

为每个组件生成测试桩，覆盖核心状态分支：

```typescript
describe('[ComponentName]', () => {
  it('renders loading state', () => { /* ... */ })
  it('renders error state', () => { /* ... */ })
  it('renders empty state', () => { /* ... */ })
  it('renders data correctly', () => { /* ... */ })
  it('handles user interaction: [主要交互]', () => { /* ... */ })
})
```

测试桩标注 `// TODO: implement` 而非空 it 块，让开发者知道要补什么。

### 第 11 步 — 接入指南与文档更新

**INTEGRATION.md** — 告诉开发者怎么把代码接进去：

```markdown
# 接入指南

## 前置条件
[列出依赖的包，带版本号]

## 安装依赖
npm install [packages]

## 接入步骤
1. 将 `src/components/[Name]/` 复制到项目 components 目录
2. 将 `src/services/[feature]Api.ts` 复制到 services 目录
3. [配置路由 / 注册全局组件 / 初始化状态管理]
4. 替换 Mock 数据：将 VITE_USE_MOCK=true 改为 false，配置真实 API baseURL

## 注意事项
[魔法值、枚举映射、特殊处理逻辑的说明]

## 已知限制
[从 quality-review.md 和 interaction-spec.md 摘取的待处理项]
```

更新 `CHANGELOG.md`，新增工程版本条目：

```markdown
## [v1.0.0-engineering] - [日期]
### 新增
- 工程级组件拆分（Phase 2）
- TypeScript 类型定义
- API 集成层（含 Mock 降级）
- 单元测试桩

### 来自 Demo 版的变更
- [列出 Demo → 工程过程中的设计调整]
```

---

## 来自 openUBMC Studio 的实践笔记

**Demo 是便宜的实验，工程代码是昂贵的承诺。**
不要在设计方向未确认时就写工程代码——Demo 阶段暴露的问题修起来是分钟级，工程代码里修是小时级。

**Hex 优于颜色名。** 始终从设计稿提取精确 hex 值。`#141420` 无歧义；「深海军蓝」则不然。

**图布局必须用布局引擎。** 永远不要硬编码图节点的 x/y 坐标。使用 ELK / dagre / ReactFlow 自动布局，或按层编程定位并记录升级路径。

**Mock 数据 = API 契约。** Demo 里的 Mock 字段形状应与真实后端返回一致。`lastLoginAt`（而非 `last_login`）命名从 Phase 1 开始就要对齐，这样 Phase 2 的 API 层是替换而非重写。

**组件拆分在写代码前确认。** 组件树比代码实现更容易改。把结构告诉用户对齐后再动手，比写完再重构省力得多。

**接入指南是工程交付的最后一公里。** `INTEGRATION.md` 写得不清楚，开发接收时会在环境配置和依赖版本上卡很久。具体写：包名 + 版本、目录位置、环境变量名称。

**暂停并确认。** Phase 1 到 Phase 2 的节点是真正的关口，不是形式。没有对齐方向就写工程代码，返工成本极高。

---

## Artifact 模板

### design-structure.md

```markdown
# [产品名称] · 设计结构文档

## 页面层级
[用 ASCII 树描述布局层级]

## 组件清单与元素描述

### [组件名]
- 类型：[容器 / 展示 / 输入 / 导航]
- 视觉：背景 #XXXXXX，圆角 Xpx，阴影 ...
- 内容层叠：[描述内部元素叠放关系]
- 间距：与上方组件间距 Xpx，内边距 Xpx
- 空状态外观：[描述]

## 非静态区域
| 区域 | 类型 | 说明 |
|------|------|------|
| ...  | 动态数据 / 条件渲染 / 动画 | ... |

## 色彩 Token
| Token | 值      | 用途 |
|-------|---------|------|
| ...   | #XXXXXX | ...  |

## 数据接口
​```typescript
interface Entity {
  id: string
  // ...
}
​```

## 数据流
[描述数据如何在页面中流动]
```

### interaction-spec.md

```markdown
# [产品名称] · 交互细化说明

## 可交互区域与交互动作

### [组件名]
| 触发方式 | 系统响应 | 状态变化 | Demo 实现程度 |
|---------|---------|---------|-------------|
| 点击    | ...     | ...     | 完整 / Mock / 占位 |

## 关键交互序列
### [交互名称]
1. 用户操作
2. 系统响应
3. 状态变化
4. 逆操作路径

## 可行性与拓展性评估
| 功能 | 当前可行性 | 拓展风险 | 说明 |
|------|-----------|---------|------|
| ...  | ✅ 完整实现 / ⚠️ 有限制 / ❌ 不可行 | 低 / 中 / 高 | ... |

## ⚠️ 待交互设计师确认
以下内容从设计稿无法确定，请设计师补充后方可进入开发：

1. [ ] [动画时长与缓动曲线]
2. [ ] [错误提示文案与出现时机]
3. [ ] [边缘状态视觉处理：空数据 / 超长文本 / 权限不足]
4. [ ] [多步骤流程的回退规则]
```

### quality-review.md

```markdown
# [产品名称] · 易用性审查报告

## 审查结论
[整体可用 / 存在若干需关注问题 / 存在影响核心流程的问题]

## 问题清单

| # | 维度 | 问题描述 | 状态 | 说明 |
|---|------|---------|------|------|
| 1 | 可读性 | ... | ✅ 已修复 / ⚠️ 已知 / ❌ 需设计师介入 | ... |

## 未覆盖的审查项
[因 Demo 性质无法验证的项目，如真实数据量下的性能表现]
```

### INTEGRATION.md（Phase 2 模板）

```markdown
# [功能名称] · 工程接入指南

## 前置条件
- Node.js >= XX
- [框架] >= [版本]
- [其他依赖]

## 安装依赖
​```bash
npm install [packages]
​```

## 文件放置
​```
your-project/
├── src/
│   ├── components/[Name]/     ← 复制此目录
│   └── services/[feature]Api.ts  ← 复制此文件
​```

## 配置
[路由注册 / 全局组件注册 / 状态初始化步骤]

## 替换 Mock 数据
将 .env 中 VITE_USE_MOCK=true 改为 false，
并配置 VITE_API_BASE_URL 指向真实后端地址。

## 注意事项
[魔法值处理 / 枚举映射 / 特殊业务逻辑]

## 已知待处理项
[从 quality-review.md 和 interaction-spec.md 摘取]
```
