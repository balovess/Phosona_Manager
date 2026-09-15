# Phosona Manager 插件架构规范

## 目标与边界

插件是受权限约束的扩展单元。当前版本完成插件目录安装、权限披露、清单发现、开发者声明、逐项授权、撤销、审计、声明式网络规则运行时，以及原始 Chrome/Firefox WebExtension 的 `native-compat` 加载。插件操作的最终判定由独立的 Rust `plugin-broker` 负责；Electron 只是受控适配层，渲染器只负责 UI。声明式插件代码不会在渲染器或 Electron 主进程中执行；原始 WebExtension 代码只在 Chromium 扩展上下文中运行，不会进入应用的 Node、Electron 或 preload 上下文。`native-compat` 不是“每个扩展 API 调用均经 Rust”的严格证明；严格运行时仍必须使用独立的 Rust/操作系统沙箱和能力代理。

## 存储位置

- 内置插件：开发环境为仓库 `plugins/<plugin-id>/`，发布包为 `resources/app.asar/plugins/<plugin-id>/`，只读并随应用发布。
- 用户插件：Electron `app.getPath("userData")/plugins/<plugin-id>/`。Windows 通常是 `%APPDATA%/phosona-manager/plugins/`，macOS 是 `~/Library/Application Support/phosona-manager/plugins/`，Linux 通常是 `~/.config/phosona-manager/plugins/`；最终以 Electron 返回的 `userData` 为准。
- 应用状态：`userData/settings.json` 保存启用列表、精确权限键和浏览器侧栏布局；插件目录不保存运行时状态。

用户通过“添加插件”选择一个单文件插件包（`.zip`、`.xpi` 或 `.crx`），不能选择解包目录。安装器先在 `userData/plugins` 下创建一次性 staging 目录，校验 ZIP/CRX 路径、压缩大小、校验和、根目录 `manifest.json`、符号链接和包结构；验证通过后才把扩展文件复制到 `userData/plugins/<plugin-id>/`，staging 目录随后删除。安装前会展示开发者、包 SHA-256 摘要和完整权限清单，并保持“禁用 + 所有权限拒绝”的安全初始状态；安装确认不会自动授予权限。相同 ID 的用户插件覆盖内置版本，用于本地更新；清单无效或包结构不安全时不复制。

## 清单格式

```json
{
  "id": "media.capture",
  "name": "Media Capture",
  "version": "1.0.0",
  "description": "Captures declared media resources.",
  "developer": {
    "name": "Example Labs",
    "website": "https://example.invalid",
    "contact": "security@example.invalid",
    "publicKey": "base64-ed25519-public-key"
  },
  "permissions": [
    {
      "id": "browser.webRequest",
      "scope": "https://media.example",
      "purpose": "Inspect media requests on the declared origin"
    },
    {
      "id": "downloads.create",
      "purpose": "Create a download after the user approves it"
    }
  ],
  "runtime": {
    "type": "declarative",
    "requestRules": [
      {
        "id": "block-media-tracker",
        "action": "block",
        "urlPattern": "https://media.example/pixel*",
        "resourceTypes": ["script", "image", "ping", "fetch", "xhr"]
      }
    ]
  }
}
```

`id` 必须匹配 `[a-z0-9][a-z0-9._-]{1,63}`。`developer.name` 必填，其他开发者字段用于审查和联系。权限可以是字符串（仅适用于不需要范围的能力）或对象；Cookie、凭据和 WebRequest 权限必须使用具体的 `http(s)` origin、用途和逐项授权。不得使用 `*`、`all`、`admin` 或自由文本扩大授权范围；未识别权限按最高风险处理。

## 权限模型

- `standard`：通知、系统托盘等低风险能力。
- `sensitive`：下载队列、浏览器读取、网络请求、文件读取等能力。
- `critical`：Cookie/凭据、文件写入、WebRequest 全量观察、进程或 Shell 等能力。

v1 能力目录为 `browser.webRequest`、`browser.filtering`、`browser.storage`、`browser.tabs`、`browser.activeTab`、`browser.scripting`、`browser.alarms`、`browser.offscreen`、`browser.userScripts`、`browser.contextMenus`、`network.connect`、`clipboard.read`、`clipboard.write`、`filesystem.read`、`filesystem.write`、`downloads.create`、`process.execute`，并保留已有的 `notifications`、`system.tray` 兼容项。权限以 `permission-id + scope` 作为精确键逐项展示，默认拒绝。每项能力可以选择“仅本次、当前会话、始终允许、不允许”；会话授权不会写入磁盘，撤销或应用重启后失效。持久授权同时绑定插件包 SHA-256 摘要，包内容变化后旧授权全部清除。Cookie、Native Messaging、Debugger 和 Proxy 在 v1 为系统永久拒绝。授予 `https://example.com` 不会授予其他 origin。高风险权限在原生授权窗口中显示风险、用途和范围。

插件是否启用与能力授权相互独立：插件可以启用，但未获授权的能力仍由 broker 直接拒绝；撤销一项权限只立即移除该项能力，不会误关插件的其他无权限功能。`process.execute` 在 v1 永久硬拒绝，任何清单或授权选择都不能改变这一规则。

能力请求必须携带 `pluginId`、包摘要、能力、操作、资源、用户发起标记和请求 ID。Rust broker 会校验当前策略版本、插件身份、声明键、授权状态和精确范围；代理不可用时全部插件能力拒绝，普通浏览器和 aria2 功能继续运行。拒绝原因写入有界审计日志，但 URL 只保留 origin，其他资源去除查询串，不记录剪贴板内容、文件内容、Cookie、Token 或私密 URL。

文件范围必须是绝对路径，不允许通配符或 `..`；broker 会阻止路径前缀逃逸，并检查符号链接和 Windows 重解析点。真实文件读写适配器接入后必须继续使用同一判定 seam，不能绕过 broker 直接把 Node 文件 API 暴露给插件。

## 分层与 IPC

- TS/Preact 只展示权限、发起用户确认和显示脱敏审计，不决定最终授权。
- Electron 主进程只做受控适配：浏览器 `webRequest` 使用已由 broker 接受的规则，页面不会获得 Node、Electron、剪贴板或文件 API。
- Rust `plugin-broker` 作为私有 stdio 子进程运行，使用行分隔 JSON 的类型化消息接收策略、判定能力请求和返回审计；不监听网络端口。策略版本和包摘要绑定，代理退出或响应异常时能力 fail-closed。
- 声明式插件包只能提交清单和规则；另有独立的 `webextension` 包类型可保留并加载原始 Manifest V2/V3 包。原始扩展代码不在渲染器、主进程或 preload 执行，也不能获得 Node/Electron 对象；但当前 Electron 适配器仍不是严格的全 API Rust 代理。

当前声明式运行时只提供 `block` 请求规则，且每条规则必须匹配已授权的 `browser.webRequest` origin；主框架导航永远不会被规则阻断。用户启用插件后，新请求立即经过规则检查，撤销权限会让对应规则失去生效资格。原始 WebExtension 则只有在必需权限均已授权后才加载，停用、撤销或摘要变化会卸载。插件页的“实际效果与权限审计”展示 broker 的判定结果和原始扩展加载状态，例如已阻止的像素请求、未授权的剪贴板请求、范围外路径拒绝和 `process.execute` 永久拒绝。声明式规则格式不翻译、不执行 uBlock 代码；uBlock 只通过原始 WebExtension 包宿主加载。

能力代理只能暴露声明的最小操作；不能暴露任意 Node、Electron、Shell、进程、文件系统路径或应用内部对象。用户拒绝后调用必须返回拒绝错误，不能通过重试、改名、伪造清单、重复 IPC 或读取插件目录绕过。`webextension` 包的可执行生命周期由独立宿主管理，并且只接受摘要和权限策略允许的包；当前宿主使用 Electron 的原生兼容适配，不应被误写成 Rust 已代理所有扩展 API。插件 JavaScript 绝不放进 Electron 主进程或 preload。

## Chrome/Firefox WebExtension 兼容边界

Chrome/Firefox WebExtension 兼容方案单独记录在 [`docs/webextension-compatibility.md`](./webextension-compatibility.md)。真正的 WebExtension 必须执行 JavaScript，不能与“插件代码永不执行”同时成立；因此安全模式继续保持声明式，脚本兼容必须作为独立的事件驱动 Lite Runtime，并通过 Rust Broker 代理全部特权能力。Electron 原生 `loadExtension` 仅属于受限开发/可信扩展模式，不作为严格安全边界。

当前版本支持原始 Manifest V2/V3 包的原生兼容加载；扩展实际可用 API 取决于 Electron 当前实现范围。严格的 Rust-mediated Lite Runtime、完整 API Shim、跨平台扩展 API 一致性和每次 API 调用级审计仍未完成。后续实现顺序和兼容性验收见 [`docs/webextension-compatibility.md`](./webextension-compatibility.md)。

## 版本与审核

插件更新必须保持 ID 不变并递增版本；新增权限或扩大 origin 范围视为高风险变更，需要重新确认。PR 应包含开发者声明、清单、权限理由、数据流说明和撤销权限后的行为。禁止插件修改 `settings.json`、访问其他插件目录或调用未声明的 Electron/Node 能力。发布前应运行 `bun run test:plugins`。
