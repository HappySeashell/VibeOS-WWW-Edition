# VibeOS 功能及软件架构说明文档

> **面向开发者**  
> VibeOS 是一个纯前端模拟操作系统。它在浏览器单页 HTML 中构建类桌面环境，并通过大语言模型动态生成应用 UI 与交互结果。本文档以 `VibeOS.html` 为事实源，说明当前源码中的模块、数据流、存储模型和扩展边界。

---

## 1. 系统概述

VibeOS 当前是一个单文件 HTML 应用，主体由三个部分组成：

- 运行时 UI 外壳：`#os-container`、`#desktop`、`#start-menu`、`#taskbar`、`#modal-container`、`#notification-container`。
- 系统应用模板：`tpl-app-settings` 和 `tpl-app-search`。
- JavaScript 运行时：窗口管理、事件委托、AI 调度、DOM patch、图片缓存、主题、设置、记忆与上下文管理。

核心交互链路是：

```text
用户操作
  -> EventDelegator 收集 actionSequence
  -> AppController 读取当前 HTML 快照
  -> PromptBuilder 拼装上下文和 UI 生成规则
  -> AIScheduler 串行调用 APIGateway
  -> AIResponseParser 提取 sys-summary/sys-call 并剥离协议标签
  -> DOMPatcher 严格按 ID patch DOM
  -> AppMemoryManager 保存应用情节记忆
  -> UIManager.handleImageHydration 挂载和缓存图片
```

---

## 2. 当前 UI 外壳

```text
#os-container
  #desktop
  #modal-container
  #notification-container
  #start-menu
    app-search
    app-settings
    #installed-apps-list
  #taskbar
    .btn-start
    #taskbar-apps
```

- 所有窗口都挂载在 `#os-container` 内。
- 用户应用窗口由 `tpl-os-window` 克隆生成，内容区 ID 为 `content-${appId}`。
- 系统应用使用预置模板，不走 AI 生成和用户应用事件委托。
- 当前仍有少量系统模板按钮使用 inline `onclick`，用户应用中的 AI HTML 不允许输出 `<script>`。

---

## 3. 核心模块

| 模块 | 职责 |
|---|---|
| `Utils` | UUID、HTML 解析、防抖、图片 prompt hash、快照净化 |
| `StorageManager` | `localStorage` 同步存储与 IndexedDB 图片 Blob 缓存 |
| `Config` | LLM、图片、系统应用、主题白名单、AI 可用组件、记忆配置 |
| `UIManager` | 设置页、应用管理、开始菜单、长期记忆列表、主题 UI、图片挂载 |
| `SettingsManager` | 世界观、主线目标、用户设定的模板、编辑和 prompt 拼接 |
| `ThemeManager` | 默认/自定义/AI 生成主题的保存和应用 |
| `APIGateway` | OpenAI 兼容、Google AI Studio、SillyTavern 三类文本模型入口 |
| `ContextStore` | 应用注册表、应用 session、旧数据迁移、快照保存 |
| `MemoryManager` | 跨应用长期记忆的手动编辑、保存、清理和追加 |
| `AIResponseParser` | 解析 AI 响应中的 `sys-summary`、`sys-call` 并返回可 patch HTML |
| `SystemStateManager` | 从运行窗口派生 `[全局系统状态]` prompt 块 |
| `AppMemoryManager` | 应用级情节记忆、分段压缩、关闭总结、墓碑清理 |
| `AppRegistry` | 统一读取原生系统应用、预置 AI 应用和用户安装应用 |
| `TaskbarManager` | 任务栏 tab、后台高亮、折叠菜单和真实时间 |
| `SystemCallExecutor` | 执行 AI 返回的受控 `sys-call` |
| `DesktopManager` | 桌面文件对象、预置应用快捷方式和文件查看窗口 |
| `DragDropManager` | 跨应用、开始菜单、桌面和窗口的拖拽投递 |
| `StorageInspector` | 估算本地存储和应用 session 占用 |
| `PromptBuilder` | 构建创建/更新 UI 的系统 prompt 与用户 prompt |
| `ComponentRuntime` | 给 AI 语义组件补运行时行为 |
| `DOMPatcher` | 严格按 ID 替换节点，允许新增 `dialog` |
| `ImageService` | LoremFlickr、NovelAI、DALL-E 占位图片生成接口 |
| `AIScheduler` | 文本任务和图片任务双队列、优先级和串行调度 |
| `AppController` | 用户应用交互的核心调度入口 |
| `EventDelegator` | 全局事件捕获、动作缓冲和用户应用事件隔离 |
| `WindowManager` | 窗口创建、关闭、最小化、最大化、置顶和加载遮罩 |
| `NotificationCenter` | 桌面右下角通知气泡 |
| `ModalManager` | 系统级 confirm/prompt/dialog |
| `AppRouter` | 系统应用启动、用户应用冷启动和 session 恢复 |

---

## 4. AI 输出协议

用户应用的 AI 响应由两部分组成：可 patch 的 HTML 节点，以及系统协议标签。

```html
<div id="some-existing-node">...</div>
<sys-summary>用户点击了登录按钮，UI 切换至主面板。</sys-summary>
<sys-call type="application/json">[]</sys-call>
```

- `sys-summary`：必须随每次 AI 响应返回，一句话描述用户动作和界面结果，进入应用级情节记忆，不进入真实 DOM。
- `sys-call`：预留未来扩展操作；当前只解析 JSON 数组并保存，不执行。
- `AIResponseParser` 会在 `DOMPatcher` 前移除协议标签，并把 calls 交给 `SystemCallExecutor`。
- 如果缺少 `sys-summary`，系统用 actionSequence 生成保守 fallback，不阻断 UI patch。

当前支持的 `sys-call`：

| 调用 | 行为 |
|---|---|
| `notify` | 弹出右下角通知，可点击打开应用并传递消息 |
| `openApp` | 唤起或打开另一个应用，并触发目标应用 AI 更新 |
| `installApp` | 虚拟安装应用，执行前确认 |
| `createDesktopItem` | 创建桌面文件对象，执行前确认 |

---

## 5. 上下文与记忆模型

当前上下文分三层：

| 类型 | 内容 | 生命周期 | 进入 prompt 的方式 |
|---|---|---|---|
| 工作记忆 | 当前 UI 快照、当前 actionSequence | 单次请求 | `[当前 UI 快照]` + `[用户动作]` |
| 应用情节记忆 | 每次 `sys-summary`、分段摘要、最近交互状态 | 应用 session 内 | `[应用情节记忆]` |
| 全局记忆 | 世界观、主线、用户设定、长期记忆、运行中应用状态 | 跨应用 | 系统 prompt 前置块 |

`Config.System.memory` 控制应用记忆策略：

```js
{
  recentEpisodeLimit: 8,
  compactEvery: 8,
  tombstoneDays: 7
}
```

`ContextStore.sessions[appId]` 当前结构：

```js
{
  htmlSnapshot,
  episodes: [{ id, at, action, summary }],
  episodeSummaries: [{ from, to, at, summary }],
  lastInteractionSummary,
  lastOpenedAt,
  updatedAt,
  closePending
}
```

旧版 `{ htmlSnapshot, history }` 会在加载时迁移：HTML 快照保留，旧 raw history 不再进入 prompt。

---

## 6. DOM Patch 策略

`DOMPatcher.applyPatch(container, html, options)` 的规则：

1. 冷启动可通过 `allowFullMount` 全量挂载窗口内容。
2. 后续更新的每个顶层节点必须有 ID。
3. 顶层节点 ID 不得等于 `content-${appId}`。
4. ID 必须在当前应用窗口内容中匹配。
5. 只有 `dialog` 可以作为新增节点追加。
6. 匹配失败不全量回退，而是发出“交互失败”通知。

这能避免 AI 输出意外替换整个应用容器，也避免旧版全量回退导致状态错乱。

---

## 7. 语义组件运行时

`ComponentRuntime` 是当前源码中的正式运行时模块。它负责：

- `tool-bar` / `tool-item` / `tool-popup`：下拉菜单展开、关闭和 hover 切换。
- `tree-view`：带子树的 `li` 自动展开/折叠。
- `tab-view`：通过 `role="tab"` 与 `aria-controls` 切换面板。
- `switch-control`：本地切换 `aria-checked`，并缓冲 change 动作。
- `segment-control`：本地单选切换并缓冲 change 动作。
- 应用内 `dialog`：自动补关闭按钮、自动 `showModal()`、取消时由 `EventDelegator` 关闭。

AI 仍然只能输出静态 HTML，组件行为由系统运行时提供。

---

## 8. 图片生命周期

图片使用：

```html
<img id="unique-img-id" class="img" data-prompt="800x600|city,night" src="">
```

生命周期：

1. AI 输出 `img[data-prompt]`。
2. 挂载前写入占位 SVG。
3. `UIManager.handleImageHydration(appId, container)` 扫描图片。
4. 先按 `Utils.hashPrompt(img.id + "|" + data-prompt)` 查 IndexedDB。
5. 命中则恢复 Blob URL。
6. 未命中则进入 `AIScheduler.enqueueImage()`。
7. 生成成功后写入 DOM、IndexedDB，并用净化后的 HTML 更新 session 快照。
8. 生成失败后写入 `data-failed-prompt`，用户点击可重试。
9. GC 会扫描当前屏幕和所有 session 快照中的图片 key，只清理彻底不再引用的 Blob。

---

## 9. 启动与恢复流程

`window.onload` 流程：

1. 从 `vibe_config` 读取保存配置。
2. 合并 `Config.LLM`、`Config.Image`、`Config.System`。
3. 强制恢复源码中的核心 `systemApps` 和 `SETTING_TEMPLATES`，防止旧缓存污染。
4. 如果存在 SillyTavern 环境函数，自动切换 `Config.LLM.apiMode = 'SILLYTAVERN'`。
5. 初始化 `ThemeManager`、`SettingsManager`、`MemoryManager`。
6. 加载 `ContextStore` 注册表和 sessions，并迁移旧 session 结构。
7. 启动 `EventDelegator`。
8. 空闲时执行 `AppMemoryManager.runIdleTombstoneSweep()`。

应用启动：

- 系统应用：直接克隆模板。
- 用户应用有 `htmlSnapshot`：恢复快照并挂载图片。
- 用户应用无快照：创建加载窗口，调用 AI 生成首屏。

---

## 10. 关闭与墓碑机制

正常关闭用户应用：

- `WindowManager.closeWindow()` 清理窗口、任务栏和图片队列。
- 如果 session 有未沉淀应用记忆，`AppMemoryManager.finalizeAppMemory(appId, "normal-close")` 后台总结。
- 总结成功后写入长期记忆，并清理应用级 `episodes` 与 `episodeSummaries`。
- 总结失败时保留应用级记忆并设置 `closePending: true`，等待后续重试。

异常刷新或直接关闭页面：

- 不在 `beforeunload` 调用模型。
- `vibe_sessions` 保留快照和应用级记忆。
- 下次打开应用时继承未关闭的上下文。

墓碑机制：

- 启动后空闲扫描 sessions。
- 超过 `Config.System.memory.tombstoneDays` 未打开且仍有应用级记忆的 session，会后台总结进长期记忆并清理。
- 总结失败则保留，等待下次空闲重试。

---

## 11. 存储模型

### LocalStorage

| Key | 内容 |
|---|---|
| `vibe_config` | LLM、Image、System 可配置字段 |
| `vibe_memory` | 跨应用长期记忆数组 |
| `vibe_current_theme` | 当前主题名 |
| `vibe_custom_themes` | 用户自定义主题 CSS |
| `vibe_registry` | 用户安装应用元信息 |
| `vibe_sessions` | 用户应用 HTML 快照和应用级情节记忆 |
| `vibe_user_settings` | 世界观、主线、用户设定 |
| `vibe_desktop_items` | 桌面文件对象和拖拽创建的内部文件 |

### IndexedDB

- 数据库：`VibeOS_ImageCache`
- Store：`blobs`
- Key：`Utils.hashPrompt(img.id + "|" + prompt)`
- Value：图片 Blob

---

## 12. 当前限制

- 项目仍是单文件 HTML，模块通过全局对象组织，未拆分构建。
- 系统模板中仍有少量 inline `onclick`，用户应用 AI HTML 仍禁止脚本。
- 关闭总结和墓碑总结依赖可用文本模型；失败会保留应用级记忆等待重试。
- `sys-call` 当前只开放通知、打开应用、安装应用和创建桌面文件四类白名单能力。
- 流式传输目前不做真实 DOM streaming patch，仍在完整响应结束后统一 patch。
- 文件导入/导出能力没有形成通用文件桥接层。

---

## 13. 任务栏、拖拽与桌面文件

- `Config.System.protectedAiApps` 注册浏览器、命令行和文件管理器。它们不可卸载，但走 AI 冷启动、更新、记忆和系统调用。
- 开始菜单和应用管理通过 `AppRegistry` 动态渲染，不再硬编码设置/查找应用。
- `TaskbarManager` 负责任务栏折叠、后台更新高亮和右侧真实时间。
- `DragDropManager` 统一包装拖拽对象为轻量 payload，支持窗口、开始菜单应用项、桌面应用图标和桌面文件。
- 拖到桌面会通过 `DesktopManager` 创建持久化内部文件；点击桌面文件会打开 AI 查看窗口。
- 命令行语义组件包含 `command-shell`、`command-output`、`command-input-row`、`command-drive`、`command-input`，Enter 会提交命令动作。

---

## 14. 扩展建议

- 新增 LLM provider：扩展 `APIGateway.providers`，并补设置 UI 选项。
- 新增图片 provider：扩展 `ImageService.providers`，实现 `doGenerate()` 和 `instruction`。
- 新增 AI 可用组件：先加入 CSS 和 `ComponentRuntime` 行为，再加入 `Config.System.allowedUIClasses`。
- 新增系统应用：扩展 `Config.System.systemApps` 并新增 `<template>`。
- 新增 `sys-call` 能力：在 `AIResponseParser` 保持只解析的基础上，增加白名单执行器，避免模型任意触发危险行为。
