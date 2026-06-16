# VibeOS 功能及软件架构说明文档

> **面向开发者**  
> VibeOS 是一个纯前端模拟操作系统，通过大语言模型动态生成应用界面与交互逻辑，被称为“AI 幻觉操作系统”。  
> 本文档详细说明其模块划分、核心接口、数据流和设计思想，以便开发者理解、调试与扩展。

---

## 1. 系统概述

VibeOS 在浏览器中构建了一个完整的类桌面环境，包含：

- 多窗口管理、任务栏、开始菜单；
- AI 驱动的 UI 生成与 DOM 补丁；
- 可插拔的文本/图像 API 后端；
- 持久化存储（LocalStorage + IndexedDB）；
- 主题/皮肤系统，支持 AI 生成主题；
- 事件记忆、用户设定等角色扮演特性。

整体采用**事件驱动 + 调度中心**的设计，用户交互被捕获为动作序列，发送给 AI 模型进行推理，返回 HTML 片段后通过 DOM 补丁器局部更新界面，避免整页刷新。

---

## 2. 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│                       Browser Window                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  #os-container (UI 沙箱)                │  │
│  │  ┌──────────┐  ┌──────────┐  ┌─────────────────────┐  │  │
│  │  │ Desktop   │  │ Taskbar   │  │ Start Menu / Modal  │  │  │
│  │  │ (窗口区)   │  │ (任务栏)  │  │ (弹出层)            │  │  │
│  │  └──────────┘  └──────────┘  └─────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────── 核心层 ──────────────────────────────┐ │
│  │  EventDelegator → AppController → PromptBuilder          │ │
│  │        │                   ↓                              │ │
│  │        │             AIScheduler                          │ │
│  │        │           /            \                         │ │
│  │        │     APIGateway      ImageService                 │ │
│  │        ↓           \            /                         │ │
│  │  WindowManager    DOMPatcher ← UIManager                  │ │
│  └──────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌──────────────── 存储与配置层 ────────────────────────────┐ │
│  │  StorageManager.Sync  |  StorageManager.BlobCache        │ │
│  │  Config  |  ContextStore  |  MemoryManager               │ │
│  │  ThemeManager  |  SettingsManager                        │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

- **UI 沙箱**：所有窗口、桌面、任务栏均在 `#os-container` 内，使用绝对定位模拟桌面。
- **事件流**：用户操作 → `EventDelegator` 缓冲为动作序列 → `AppController.dispatchIntent()` → 调用 LLM → `DOMPatcher.applyPatch()` 更新界面。
- **调度**：`AIScheduler` 统一管理文本与图像生成任务的优先级与并发。
- **存储**：配置、会话、记忆、主题通过 `StorageManager` 持久化；图像 Blob 缓存于 IndexedDB。

---

## 3. 核心模块详细说明

### 3.1 基础工具类 `Utils`

提供通用无状态函数。

```js
const Utils = {
  generateUUID()       // 生成 id-xxxxxxxxx 格式的随机 ID
  parseDOM(htmlString) // HTML字符串 → DOM节点数组（借助 DOMParser）
  debounce(func, wait) // 防抖函数
  hashPrompt(str)      // 对字符串进行简单哈希，返回 'img_xxx' 格式
  getPurifiedHTML(container) // 克隆容器并清除 img 的 src（保留 data-prompt）
}
```

- `getPurifiedHTML` 用于存储和发送给 AI 的 UI 快照，避免 Blob URL 污染。

---

### 3.2 统一存储管理器 `StorageManager`

封装两套存储子系统。

#### 3.2.1 同步配置存储 `StorageManager.Sync`

基于 `localStorage`，方法：

| 方法 | 参数 | 说明 |
|------|------|------|
| `load(key, defaultValue)` | `string`, `any` | 读取并 JSON.parse，若无返回默认值 |
| `save(key, value)` | `string`, `any` | 序列化为 JSON 并存储，异常时静默失败 |
| `remove(key)` | `string` | 删除键 |
| `clearData(keys)` | `string[]` | 批量删除多个键 |

#### 3.2.2 异步瞬态图像缓存 `StorageManager.BlobCache`

基于 `IndexedDB`，用于缓存生成的图片 Blob，避免重复请求。

| 方法 | 说明 |
|------|------|
| `init()` | 初始化数据库，确保 object store 存在 |
| `put(cacheKey, blob)` | 存入 Blob（key 为图片哈希） |
| `get(cacheKey)` | 获取 Blob 或 null |
| `gc(activeKeys)` | 垃圾回收，删除不在 `activeKeys` 中的缓存 |
| `clearAll()` | 清空所有图片缓存 |

**使用场景**：应用关闭或刷新后，重新打开的窗口从 IndexedDB 恢复图片，保持视觉效果。

---

### 3.3 全局配置中心 `Config`

存储所有运行时可配置项，作为单一数据源。

```js
Config.LLM        // 文本模型相关
Config.Image      // 生图模型相关
Config.System     // 系统级配置（允许的 CSS 类、系统应用、预设模板等）
Config.StorageKeys  // 所有 localStorage 键列表，用于一键重置
Config.UIBindings   // UI 控件 ID 与 Config 字段的映射表
```

- `Config.System.allowedUIClasses`：定义 AI 可以使用的 HTML 标签与 CSS 类名白名单。
- `Config.System.allowedThemeTokens`：主题变量白名单。
- `Config.System.systemApps`：内置系统应用（设置、查找应用）。
- `Config.System.SETTING_TEMPLATES`：世界观、主线、用户设定的预设模板。

**重要**：系统启动时会从 `localStorage` 恢复 `Config`，但会强制保留 `systemApps` 等核心字段，防止旧缓存覆盖。

---

### 3.4 UI 管理器 `UIManager`

负责刷新各类设置、列表、主题的 UI 状态。

```js
UIManager.refreshSettingsUI()         // 更新世界观/主线/用户设定的下拉和文本框
UIManager.renderAppManagementList(keyword) // 渲染“应用管理”列表，支持搜索过滤
UIManager.renderStartMenuApps()       // 更新开始菜单中的用户安装应用列表
UIManager.renderMemoryList()          // 更新长期记忆编辑列表
UIManager.editMemory(index)           // 进入编辑模式
UIManager.cancelMemory(index)         // 取消编辑
UIManager.saveMemory(index)           // 保存记忆项
UIManager.refreshThemeUI()            // 更新主题选择器与 CSS 编辑器
UIManager.loadSettingsToUI()          // 将 Config 中的值填入设置界面控件
UIManager.handleImageHydration(appId, container) // 为新打开/恢复的应用挂载图片（从缓存恢复或加入生成队列）
```

`handleImageHydration` 是应用加载后的重要步骤，它：
- 扫描当前容器中所有 `img[data-prompt]`；
- 优先从 `BlobCache` 恢复 Blob URL；
- 若无缓存则插入占位 SVG 并加入 `AIScheduler` 生成队列；
- 同时触发全局图片垃圾回收，收集当前活跃的图片 key 集合。

---

### 3.5 设置管理器 `SettingsManager`

管理用户的世界观、主线目标和用户设定。

```js
SettingsManager.state = {
  worldview: { current: '无', values: {...} },
  mainQuest: { current: '无', values: {...} },
  userProfile: { current: '无', values: {...} }
}
```

| 方法 | 说明 |
|------|------|
| `init()` | 从 `StorageManager.Sync` 加载保存的设定，合并预设模板 |
| `save()` | 持久化当前设定 |
| `applySelected(cat)` | 应用下拉选择，更新文本框并保存 |
| `saveEdited(cat)` | 编辑文本框内容后保存 |
| `getCombinedSettingString()` | 返回格式化的设定文本块，用于拼接到 AI 系统提示词 |

---

### 3.6 主题管理器 `ThemeManager`

支持自定义 CSS 皮肤，并可通过 AI 生成新主题。

```js
ThemeManager.themes       // { themeName: cssString }
ThemeManager.currentTheme // 当前主题名
ThemeManager.templateCSS  // 生成新主题时的模板
```

| 方法 | 说明 |
|------|------|
| `init()` | 加载保存的主题信息，应用当前主题 |
| `applyTheme(name)` | 激活指定主题，动态注入 `<style>` |
| `saveThemes()` | 保存当前主题名和自定义主题到 localStorage |
| `addAndApplyTheme(name, css)` | 添加新主题并立即应用 |
| `applySelected()` | 从下拉框读取并应用主题 |
| `saveEdited()` | 保存当前编辑的主题 CSS |
| `createNew()` | 弹出输入框，基于模板创建新主题 |
| `deleteCurrent()` | 删除当前主题（默认主题不可删） |
| `generateAITheme(event)` | 调用 LLM 根据用户描述生成完整 CSS，并添加为新主题 |

生成的 CSS 必须覆盖 `#os-container` 内的所有设计令牌变量，并重塑指定选择器的样式，但禁止改变布局、z-index 等关键属性，以保持系统功能正常。

---

### 3.7 API 桥接层 `APIGateway`

统一调用多种 LLM 后端。

```js
APIGateway.providers = { OPENAI, GOOGLE, SILLYTAVERN }
APIGateway.callModel(systemPrompt, userPrompt, onChunk, onComplete, onError)
```

**每个 Provider 实现**：
```js
{
  name, defaultUrl, defaultModel, fields: ['url','key','model','stream'],
  request: async (systemPrompt, userPrompt, config, onChunk, onComplete, onError)
}
```
- `config` 为 `Config.LLM` 的当前值。
- 若提供 `onChunk`，则启用流式传输（OpenAI 用 SSE，Google 用 `alt=sse`，SillyTavern 用全局事件）。
- `onComplete` 最终接收完整响应文本。
- `onError` 处理异常，通常会通过 `NotificationCenter` 提示用户。

**扩展新 Provider**：只需在 `providers` 中添加类似结构的对象，并在设置 UI 的 `config-api-mode` 下拉选项中添加对应项。

---

### 3.8 上下文管理器 `ContextStore`

管理用户安装的应用和应用内的会话历史。

```js
ContextStore.registry  // { appId: { name, description, icon, publisher, createdAt } }
ContextStore.sessions  // { appId: { htmlSnapshot, history: [...] } }
```

| 方法 | 说明 |
|------|------|
| `registerApp(appId, name, desc, icon, publisher, createdAt)` | 注册新应用，更新开始菜单 |
| `saveSession(appId, htmlSnapshot, intentSequence, aiResponse)` | 保存最新快照，追加对话历史（最多保留6轮） |
| `updateSnapshot(appId, htmlSnapshot)` | 仅更新 HTML 快照，不改变对话 |
| `getSessionString(appId)` | 获取格式化的对话历史字符串，用于提示词 |
| `loadRegistry()` / `saveRegistry()` | 从/到 localStorage 读写注册表 |
| `loadSessionsFromStorage()` / `saveSessionsToStorage()` | 读写会话数据 |
| `uninstallApp(appId)` | 卸载应用，清理会话、窗口，并提示用户确认 |

**会话历史格式**：`history` 为数组，包含 `{role:'user'/'assistant', content}`，用于提供给 AI 的上下文。

---

### 3.9 记忆管理器 `MemoryManager`

从操作日志中自动提取关键事件，形成长期记忆。

```js
MemoryManager.longTermMemory   // string[]
MemoryManager.interactionCounter // 累计交互次数，每5次触发一次总结
```

| 方法 | 说明 |
|------|------|
| `init()` | 从 localStorage 加载记忆 |
| `save()` | 持久化 |
| `addNew()` | 新增一条空白记忆，并自动进入编辑 |
| `checkAndSummarizeMemory(appId, actionSequence, aiResponse)` | 每5次交互调用 AI 提取事件总结；若为有效事实则插入记忆（最多10条） |
| `updateItem(index, newValue)` | 编辑记忆，空内容则删除 |
| `deleteItem(index)` | 删除指定记忆 |
| `clearAll()` | 清空所有记忆（需确认） |

---

### 3.10 提示词工厂 `PromptBuilder`

根据应用状态与系统设定，为 LLM 构建系统提示词和用户提示词。

```js
PromptBuilder.buildSystemPrompt(appId)  // 返回完整系统提示词
PromptBuilder.buildUserPrompt(domSnapshot, actionSequence) // 返回用户提示词
```

系统提示词包含：
- 用户设定（世界观、主线、用户档案）
- 长期记忆
- 应用元信息
- 允许的 HTML 标签和 CSS 类白名单
- 生成规则（禁止 Markdown、必须包含 ID、图片占位格式等）

用户提示词包含当前 UI 快照和动作序列。

**首次启动 vs 后续操作**：冷启动时要求生成丰富界面；后续更新则引导 AI 输出局部 DOM Diff。

---

### 3.11 DOM 补丁器 `DOMPatcher`

将 AI 返回的 HTML 片段智能合并到现有容器中，避免全量替换破坏状态。

```js
DOMPatcher.applyPatch(containerElement, htmlString, options)
```

**补丁策略**（按优先级）：
1. **精确 ID 匹配**：若新节点的 id 与容器内某元素相同，则替换该元素（或容器自身）。
2. **图片 Prompt 宿主匹配**：若新节点无 ID 但包含 `img[data-prompt]`，则在旧 DOM 中寻找同 prompt 的图片，向上回溯至第一级子节点进行替换。这是为了处理 AI 可能不生成 ID 的情况。
3. **全量回退**：以上均未匹配时，清空容器并整体替换。

`options.onBeforeMount` 允许在挂载前对每个新节点做预处理（如给图片插入占位 SVG）。

---

### 3.12 图片服务 `ImageService`

负责图像生成与恢复，支持多种后端。

```js
ImageService.providers = { MOCK, NOVELAI, DALLE }
ImageService.getCurrentInstruction()  // 返回当前模式下的提示词填写说明
ImageService.generate(prompt, signal) // 根据 prompt 生成图片，返回 Blob 或 url 字符串
ImageService._createFallbackSVG(text) // 生成占位 SVG
ImageService._extract(buffer)         // 原生解压 ZIP 并提取图片（用于 NovelAI）
```

**Provider 实现**：
```js
{
  defaultUrl, instruction, // 提示词说明
  doGenerate: async (prompt, config, service) => { ... }
}
```
- `MOCK`：使用 LoremFlickr 占位图，支持 `宽x高|关键词` 格式。
- `NOVELAI`：调用 NovelAI 生图 API，自动处理 ZIP 解压，支持 NSFW 参数，且包含复杂的 prompt 合并和并发控制。
- `DALLE`：预留接口，返回占位图。

`ImageService.generate` 会被 `AIScheduler` 调用，自动应用当前 `Config.Image` 配置。

---

### 3.13 AI 调度中心 `AIScheduler`

统一管理文本推理和图片生成任务的优先级与串行执行，防止并发冲突。

```js
TASK_PRIORITY = { URGENT: 0, NORMAL: 5, BACKGROUND: 10 }
AIScheduler {
  enqueueText(taskFn, priority)   // 文本任务进入队列，按优先级串行执行
  enqueueImage(appId, container, imgNode, prompt, priority) // 图片任务入队，自动去重和优先级排序
  clearImagesForApp(appId)        // 取消指定应用的所有待生成图片任务
}
```

- **文本通道**：绝对串行，每个任务完成后等待 100ms 避免速率限制。
- **图片通道**：独立串行，生成间隔 1500ms；任务执行前检查 DOM 节点是否仍在文档中，若已移除则跳过。
- 图片生成成功后，将 Blob 存入 `BlobCache`，并立即更新 `ContextStore` 快照（净化 HTML）。
- 图片生成失败时，原地显示错误占位 SVG，并保留 `data-failed-prompt` 支持点击重试。

---

### 3.14 应用控制器 `AppController`

核心业务逻辑入口：接收用户动作序列，触发 AI 推理并更新界面。

```js
AppController.dispatchIntent(appId, container, actionSequence)
```

流程：
1. 提取容器净化后的 HTML 快照。
2. 显示加载动画。
3. 构造提示词 → 送入 `AIScheduler.enqueueText`（最高优先级）。
4. 获得 AI 响应后，隐藏加载动画。
5. 调用 `DOMPatcher.applyPatch` 更新 DOM，同时预处理图片占位。
6. 保存会话快照和对话历史。
7. 触发记忆总结 (`MemoryManager.checkAndSummarizeMemory`)。
8. 执行图片挂载 (`UIManager.handleImageHydration`)。

---

### 3.15 事件委托与行为缓冲池 `EventDelegator`

监听整个 `#os-container` 的交互事件，将用户操作转换为结构化动作序列。

```js
EventDelegator.init()  // 绑定全局事件监听
```

**监听事件**：
- `input`、`change`、`focus` → `handlePassiveAction`：不立即触发推理，而是缓冲动作（如输入内容、改变选项）。
- `click`、`keydown Enter` → `handleActiveAction`：作为主动作触发，合并之前的缓冲动作，一起传递给 `AppController`。

**关键逻辑**：
- 自动为没有 `id` 的可交互元素补充 `id`（`auto-id-xxx`）。
- 过滤系统应用（settings/search）和窗口 chrome 的事件，避免不必要推理。
- 支持点击失败图片重试：将 `data-failed-prompt` 转为 `data-prompt` 并重新加入生成队列。

**动作缓冲**：
- 每个应用维护一个 `actionBuffers[appId]` 数组。
- 被动动作累积，主动作触发时取出整个数组，与当前动作合并，然后清空缓冲。
- 动作格式：`{ type, targetId, value }`（value 仅对 input/change 有效）。

---

### 3.16 窗口管理器 `WindowManager`

管理应用窗口的生命周期、拖拽、最小化、最大化。

```js
WindowManager.createWindow(appId, title, contentHTML, icon)
WindowManager.closeWindow(winId)
WindowManager.toggleMinimize(winId, forceShow)
WindowManager.toggleMaximize(winId)
WindowManager.bringToFront(win)  // 调整 z-index
WindowManager.showLoading(appId) // 显示全局加载条（位于窗口外壳，不污染内容区）
WindowManager.hideLoading(appId)
```

- 窗口 DOM 结构基于 `<template id="tpl-os-window">` 克隆。
- 拖拽实现基于 `mousedown/mousemove/mouseup`，最大化时禁止拖拽。
- 最小化通过改变 `display` 属性实现，并同步任务栏 Tab 样式。
- 关闭窗口时自动取消该应用所有待生成图片任务。

---

### 3.17 通知中心 `NotificationCenter`

在桌面右下角显示气泡通知。

```js
NotificationCenter.show({ title, message, type, duration })
```
- `type`：`info` / `success` / `warning` / `error`，影响图标和样式。
- 鼠标悬停暂停自动消失计时器。
- 支持手动关闭按钮，淡出动画。

---

### 3.18 模态弹窗管理器 `ModalManager`

提供确认框、提示输入框等模态交互。

```js
ModalManager.openModal({ title, bodyHtml, buttons })  // 自定义按钮
ModalManager.closeModal()
ModalManager.confirm(title, message)   // 返回 Promise<boolean>
ModalManager.prompt(title, message, defaultValue) // 返回 Promise<string | null>
```

`buttons` 结构：`[{ text, type ('btn-primary' etc.), onClick(container) }]`。

---

### 3.19 应用路由器 `AppRouter`

负责启动应用（系统或用户安装），区分冷启动和会话恢复。

```js
AppRouter.launchApp(appId)
```

- 系统应用：直接克隆对应 `<template>` 内的 HTML，打开窗口并触发特殊初始化（如设置应用调用 `loadSettingsToUI`）。
- 用户应用：
  - 若有 `ContextStore.sessions[appId]` 快照，直接恢复窗口和内容，然后调用 `handleImageHydration` 恢复图片。
  - 否则进入冷启动流程：创建空白窗口，调用 AI 生成首屏界面。

冷启动过程：
1. 创建窗口，内容为“初始化中...”。
2. 构造系统提示词，送入 `AIScheduler.enqueueText`（`URGENT`）。
3. AI 返回 HTML 后，用 `DOMPatcher` 替换内容，预处理图片占位。
4. 保存净化后的快照，挂载图片。

---

## 4. 关键流程

### 4.1 系统启动 (`window.onload`)

1. 从 `localStorage` 加载 `vibe_config`，合并到 `Config`（保护核心字段）。
2. 检测 SillyTavern 环境（`window.generateRaw` 等）并自动切换 API 模式。
3. 初始化 `ThemeManager`、`SettingsManager`、`MemoryManager`。
4. 加载 `ContextStore`（注册表与会话）。
5. 启动 `EventDelegator` 全局事件监听。

### 4.2 用户交互与 AI 响应流程

```
用户操作 (click/input)
  → EventDelegator 收集动作序列
  → 主动作触发 AppController.dispatchIntent()
    → 提取 DomSnapshot
    → PromptBuilder 构造系统/用户提示词
    → AIScheduler.enqueueText (优先级URGENT)
    → APIGateway.callModel → LLM 返回 HTML
    → DOMPatcher.applyPatch 更新界面
    → 保存会话、触发记忆总结、图片挂载
```

### 4.3 图片生命周期

1. **占位**：AI 生成 `<img data-prompt="..." src="">`，`onBeforeMount` 自动替换 `src` 为占位 SVG。
2. **加入队列**：`UIManager.handleImageHydration` → `AIScheduler.enqueueImage`。
3. **生成**：`ImageService.generate` → 根据配置调用后端 → 获得 Blob/URL。
4. **缓存**：成功后将 Blob 存入 `BlobCache`，更新 DOM 的 `src`，同时更新快照（净化后）。
5. **失败重试**：显示失败占位 SVG，用户点击后重新加入队列。
6. **垃圾回收**：每次打开应用或关闭时，收集当前活跃图片 key，清除无引用缓存。

---

## 5. 数据存储模型

### 5.1 LocalStorage

| 键 | 类型 | 说明 |
|----|------|------|
| `vibe_config` | object | 序列化的 `Config.LLM` + `Config.Image` + 部分 `Config.System` |
| `vibe_memory` | string[] | 长期记忆条目 |
| `vibe_current_theme` | string | 当前主题名称 |
| `vibe_custom_themes` | object | `{ themeName: css }` |
| `vibe_registry` | object | 用户安装的应用元数据 `{ appId: {name, desc, icon, publisher, createdAt} }` |
| `vibe_sessions` | object | 应用会话 `{ appId: { htmlSnapshot, history } }` |
| `vibe_user_settings` | object | `SettingsManager.state` 的序列化 |

### 5.2 IndexedDB (`VibeOS_ImageCache`)

- 数据库：`VibeOS_ImageCache`
- 对象存储：`blobs`
- 键：`Utils.hashPrompt(img.id + '|' + prompt)` 计算出的字符串
- 值：图片 Blob

---

## 6. 开发扩展指南

### 6.1 添加新的 LLM Provider

1. 在 `APIGateway.providers` 中添加新对象，实现 `request` 方法。
2. 在 `Config.UIBindings` 的 `config-api-mode` 选项中增加相应值。
3. 在设置 UI 的 `elMode.onchange` 中添加动态显示/隐藏字段的逻辑。

### 6.2 添加新的图片生成后端

1. 在 `ImageService.providers` 中添加对象，实现 `doGenerate`。
2. 更新 `Config.Image` 相关默认配置，并在设置界面 `config-img-api` 下拉增加选项。
3. 若需要额外配置项，扩展 `Image` 节并同步 UI 控件。

### 6.3 注册新系统应用

1. 在 `Config.System.systemApps` 中添加条目，指定 `templateId`。
2. 在 HTML 中创建对应的 `<template>` 并设置内部 HTML。
3. 如需在开始菜单中显示，可修改 `#installed-apps-list` 的渲染逻辑。

### 6.4 扩展 AI 可用的 UI 组件

1. 在 `Config.System.allowedUIClasses` 中添加新的标签/类组合，并说明用途。
2. 在全局 CSS 中定义好样式（位于 `:root` 外的 `[data-theme="default"]` 块或主题 CSS 中）。
3. 更新 `PromptBuilder.buildSystemPrompt` 中的组件列表（自动从配置读取，无需修改提示词代码）。

### 6.5 自定义主题

- 手动编辑：在设置 → 个性化 → 主题管理中创建新主题，编写覆盖 `#os-container` 及指定选择器的 CSS。
- AI 生成：输入描述，系统调用 LLM 生成完整主题 CSS 并自动应用。
- 主题变量完整列表见 `Config.System.allowedThemeTokens`。

---

## 7. 注意事项

- **状态一致性**：所有 DOM 修改必须通过 `DOMPatcher` 或系统窗口管理器，避免直接操作破坏事件绑定。
- **ID 重要性**：AI 生成的界面中每个交互元素必须有唯一 `id`，否则事件系统无法定位目标。
- **图片净化**：存储快照时务必使用 `Utils.getPurifiedHTML` 清除 Blob URL，避免 localStorage 撑爆和跨会话污染。
- **加载状态隔离**：`WindowManager.showLoading` 将加载条挂载在窗口外壳而非内容区，防止 AI 更新内容时意外清除加载动画。
- **事件防火墙**：`EventDelegator` 忽略来自 `#modal-container` 和系统应用内部的事件，避免干扰系统功能。
- **并发控制**：文本请求严格串行，图片生成有独立队列和冷却时间，防止 API 速率限制。
- **IndexedDB GC**：`handleImageHydration` 会触发全局垃圾回收，但若直接刷新页面而未打开任何窗口，缓存可能残留，可通过“恢复出厂设置”清除。

---

## 8. AI 语义组件与严格 Patch 规范

### 8.1 AI 应用推荐结构

AI 首屏应优先输出：

```html
<tool-bar id="app-toolbar">...</tool-bar>
<app-layout id="app-main">...</app-layout>
<status-bar id="app-status">...</status-bar>
```

- `tool-bar` 与 `status-bar` 是应用外壳，位于 `app-layout` 外。
- `app-layout` 是主内容区，系统预置 `flex: 1`、纵向布局和 `overflow: hidden`。
- 后续交互默认只更新 `app-layout` 内最深层动作相关节点；除非动作明确影响工具栏或状态栏，否则不要返回应用外壳。

### 8.2 语义组件运行时

`ComponentRuntime` 统一管理 AI 生成 UI 的预置行为：

- `tool-bar` / `tool-item` / `tool-popup` / `tool-row` / `tool-divider` / `tool-spacer`：工具栏与下拉菜单。
- `tree-view`：树状菜单展开/折叠。
- `tab-view`：基于 `role="tablist"`、`role="tab"`、`role="tabpanel"` 的标签页切换。
- `dialog`：应用内弹窗自动打开。

AI 仍然不能输出 `<script>` 或 JavaScript，所有组件行为都由系统运行时提供。

### 8.3 样式边界

- 按钮、输入框、下拉框、文本框、图片、进度条、菜单行、状态字段等实体控件不承担外边距。
- 布局、间距、滚动、分栏由外层容器或受限 inline style 控制。
- AI 允许 inline style 处理布局属性，但不得内联修改颜色、背景、边框颜色、阴影、字体、圆角、层级或绝对/固定定位等视觉主题属性。
- 主内容滚动区域应使用 `flex:1; min-height:0; overflow:auto` 这类布局属性表达，不使用系统设置专用的滚动限高样式。

### 8.4 DOM Patch 策略

`DOMPatcher` 后续交互采用严格 ID 匹配：

1. 冷启动允许全量挂载。
2. 后续更新必须返回顶层带 ID 的节点。
3. 只在当前应用 `content-${appId}` 内查找匹配 ID。
4. 匹配失败时显示交互失败提示，不再通过图片 prompt 宿主匹配或全量替换兜底。

### 8.5 Dialog 归属

- `#modal-container dialog` 是系统弹窗，不触发 AI 行为。
- `.window-content dialog` 是 AI 应用弹窗。
- AI 只输出普通 `<dialog id="...">`，不需要写系统属性。
- 应用弹窗内的输入/选择只缓冲动作；底部确认/应用按钮触发一次 AI 更新；取消/关闭只关闭弹窗。

### 8.6 冷启动流式预览

即使开启流式传输，用户应用冷启动也不会把半截 HTML 写入真实 DOM。系统只在完整响应结束后正式挂载 HTML、初始化组件、挂载图片并保存快照。后续交互 diff 同样不做真实 DOM 流式 patch。

---

## 9. 总结

VibeOS 是一个高度模块化、可扩展的前端伪操作系统沙箱。它巧妙地将 LLM 作为“应用生成引擎”，通过严格的事件驱动、DOM 补丁和队列调度，实现了流畅的类桌面交互体验。开发者可在此基础上快速定制后端、UI 组件、主题和系统行为，构建独特的 AI 原生交互原型。
