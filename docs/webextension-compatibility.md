# Chrome/Firefox WebExtension 兼容架构

## 目标

在保持 iOS 风格权限代理和“默认拒绝”的前提下，支持把原始 Chrome/Firefox WebExtension 包作为插件加载。uBlock 必须以它自己的扩展包运行；把过滤列表翻译成内部规则只能算兼容降级，不能算“支持 uBlock”。

必须先区分两种能力：

- **声明式插件**：只提交清单和规则，不执行插件 JavaScript。它是 Phosona 自有的受限规则格式。
- **WebExtension 扩展**：扩展本身必须执行 JavaScript、Service Worker、Popup 或 Content Script。这是独立的可执行运行时，不能伪装成声明式插件。

因此，“支持任意 Chrome/Firefox 扩展”和“完全不执行插件代码”不能同时成立。声明式插件继续使用严格 Rust 代理；需要执行脚本的原始扩展进入单独的原生兼容适配，并接受更严格的包审核、权限披露和能力门控。它不会被标记为严格 Rust 代理运行时。

## 运行时结构

当前原始扩展使用独立的 native-compat 适配层；Phosona 只负责包校验、权限策略、加载和卸载，扩展的 background page、Service Worker、Content Script 和 Popup 仍由 Electron 的 Chromium 扩展宿主执行：

```text
原始 WebExtension 包
          │
          ▼
Manifest/包摘要/权限披露
          ▼
Rust Plugin Broker
          │  仅允许已启用且必需权限已授权的包
          ▼
WebExtension Supervisor
          │  绑定持久 browser session，负责加载/卸载
          ▼
Electron Chromium Extension Host
          │  执行原始扩展代码
          └── 原始 background/content/popup 资源
```

核心原则：

1. 当前 Supervisor 只同步授权包的加载状态；background page/Service Worker 的具体生命周期由 Chromium 管理。
2. Rust Broker 保存身份、摘要、声明和授权状态；撤销、停用或摘要变化会卸载原始扩展。
3. 原生兼容模式不能证明每一次原生扩展 API 调用都经过 Rust Broker；因此始终显示为 `native-compat`。
4. 每个扩展拥有独立身份、独立 browser session 扩展注册和独立权限策略。
5. Broker 不可用时，扩展不会加载或会被卸载；普通网页、浏览器导航和 aria2 不受影响。

严格的 WebExtension Lite Runtime 才采用“按事件启动、空闲挂起、API Shim 代理”的 Service Worker 方案，下面的生命周期和 API Shim 小节是后续实现契约，不是当前 Electron 原生适配器已经提供的能力。

## 三种模式

| 模式 | 执行 JS | 适用对象 | 安全边界 |
| --- | --- | --- | --- |
| 安全模式 | 否 | 不可信插件、Phosona 声明式插件 | 只有清单和声明式规则 |
| WebExtension Lite | 是，按事件启动 | 后续的审核后扩展 | 独立运行时 + Rust API Shim（尚未作为 Electron 现成能力宣称） |
| 原生兼容模式 | 是，Chromium 托管 | 当前用于原始 uBlock/兼容扩展 | 通过权限门控加载，但不保证全部原生 API 调用经过 Broker |

原生兼容模式不能冒充严格模式。Electron 的扩展能力只是 Chrome API 的子集，并且原生加载接口只接受解包目录；我们的安装器因此先接收单文件包，再把通过校验的内容放入应用管理的插件目录。它不能证明任意扩展都能被我们的权限代理完全控制。当前实现只在包启用且所有必需权限已授权时调用 Electron 原生加载接口，并在停用、撤销、摘要变化时卸载。真正要求“每个扩展 API 调用都经过 Rust”的 Lite Runtime 仍需要独立的 API Shim/宿主，不能把 `loadExtension` 当作这条安全证明。

## 扩展包安装

安装流程统一为：

```text
用户选择的单文件扩展包（.zip / .xpi / .crx）
  ↓
在 userData/plugins 下创建一次性 staging 目录并安全解压
  ↓
检查 manifest.json、压缩/解压大小、路径、校验和、符号链接和文件数量
  ↓
计算包 SHA-256 摘要
  ↓
复制扩展文件到 userData/plugins/<plugin-id>/（不保存用户选择路径）
  ↓
编译为 Phosona 权限策略
  ↓
显示完整披露和风险
  ↓
用户确认安装，但所有权限仍默认拒绝
```

包摘要必须绑定权限策略。扩展文件、清单、Service Worker、Content Script 或规则文件发生变化后，旧的持久授权全部失效并重新请求。

安装器不执行包内代码，不允许包内符号链接、路径遍历、远程脚本和安装脚本。ZIP、XPI 和 CRX 只作为单文件输入；CRX 头会被剥离后按 ZIP 校验，包签名不在当前版本验证，用户仍会看到摘要、来源和权限披露。用户选择路径不会成为运行时路径，运行时只加载应用管理的 `userData/plugins/<plugin-id>/`。native-compat 扩展还需要持久 browser session；开启“会话级网站存储”时，普通网页仍可使用，但原始扩展会保持未加载并显示原因。

## Manifest 权限映射

| Chrome/Firefox 声明 | Phosona 能力 | 默认处理 |
| --- | --- | --- |
| `host_permissions` | 扩展 host scope | 参与原始扩展的页面/请求权限披露；native-compat 不把它伪装成整机网络授权 |
| `webRequest` | `browser.webRequest` | 原始包在 native-compat 中保留原 API；严格 Lite Runtime 需要 Broker API Shim |
| `declarativeNetRequest` | `browser.filtering` | 原始包交给 Chromium 扩展宿主；不翻译为 Phosona 自有规则 |
| `clipboardRead` | `clipboard.read` | 与写入分开，并要求用户动作 |
| `clipboardWrite` | `clipboard.write` | 与读取分开授权 |
| `downloads` | `downloads.create` | 只允许用户批准的下载 |
| `storage` | 扩展独立存储 | 不得访问 `settings.json` 或其他扩展目录 |
| `scripting` | 页面脚本注入 | 必须同时满足 host scope 和注入规则 |
| `cookies` | 暂不支持 | v1 永久拒绝 |
| `nativeMessaging` | 暂不支持 | 禁止连接外部程序 |
| `debugger`、`proxy` | 暂不支持 | v1 永久拒绝 |

`<all_urls>`、`*`、`all` 等不能自动转换为整机或全网络授权。无法细化为可审计范围的声明默认拒绝。

### 当前 Electron 适配范围

当前版本实际使用 Electron 44 的 Chromium 扩展宿主。已在桌面 smoke 中验证原始 Manifest V2 background 脚本、content script、`webRequest` 阻断以及授权后的卸载；加载器不会改写这些脚本。Electron 官方只承诺扩展 API 的子集，因此插件页会把未列入宿主支持清单的 Manifest 权限标成兼容性提示。当前 uBlock Chromium 清单中的 `contextMenus`、`privacy`、`webNavigation` 等权限会出现这样的提示；这不影响“原包被加载”的事实，但不应描述为完整 Chrome/Firefox 兼容。

这也是为什么系统同时保留 `brokered` 与 `native-compat` 两个状态：前者表示声明式能力由 Rust Broker 逐请求判定，后者表示原始扩展代码在 Chromium 扩展上下文中运行并经过安装、启用和权限门控，但 Electron 没有提供把所有原生扩展 API 调用重新接到 Rust 的接口。

## Service Worker 生命周期（Lite Runtime 规划）

严格 Lite Runtime 的 Supervisor 规划维护以下状态。当前 native-compat Supervisor 只提供 `not-loaded`、`loaded`、`failed` 和 `unavailable` 的包级状态，并把脚本生命周期交给 Electron：

```text
installed → stopped → starting → running → suspended
                         │            │
                         └── failed   └── revoked
```

未来 Lite Runtime 的启动顺序固定为：

1. 检查扩展已启用。
2. 读取当前包摘要和策略版本。
3. 确认 Broker 健康并加载该扩展的最小策略。
4. 创建无 Node、无 Electron、无应用对象的隔离 JS 上下文。
5. 注入 API Shim 和事件队列。
6. 执行扩展包内的 Service Worker 脚本。
7. 事件处理结束后进入空闲计时；超时则停止上下文。

未来 Lite Runtime 的每个事件都应有最大执行时间、最大内存、最大消息大小和最大并发数。当前 native-compat 模式不宣称具备这些限制。

当前实现撤销权限时卸载原始扩展；Lite Runtime 还必须取消待处理请求、停止相关上下文并递增策略版本。恢复后的上下文必须重新读取策略，不能复用撤销前的授权结果。

## API Shim 与 Broker（Lite Runtime 规划）

严格 Lite Runtime 中，扩展看到的是受限的 `browser.*` / `chrome.*` 接口；它看不到 Electron 对象、Node 模块、文件句柄、Socket 或内部 IPC。当前 native-compat 扩展由 Chromium 原生扩展上下文提供这些 WebExtension API，不把每次调用转换成下面的 Broker 请求。

每次特权操作都转换为：

```text
CapabilityRequest {
  requestId
  pluginId
  packageDigest
  capability
  operation
  resource
  userInitiated
}
```

Rust Broker 检查：

- 扩展身份和包摘要；
- Manifest 是否声明能力；
- 用户是否授予当前 grant mode；
- origin、端口、tab、frame 和文件范围；
- 当前策略版本；
- 是否满足用户动作要求；
- v1 系统永久拒绝规则。

在 Lite Runtime 中，通过检查后 Electron 适配器才执行实际动作。native-compat 只在包级加载前检查这些条件，不能把 `loadExtension` 当成逐调用授权证明。

## Content Script（Lite Runtime 规划）

Content Script 能看到网页 DOM，因此不属于低风险能力。严格 Lite Runtime 的注入必须同时满足：

- 扩展拥有匹配的 host scope；
- 当前页面 origin 匹配 `matches`；
- 页面不是浏览器内部页、扩展页或受保护页面；
- 注入动作经过 Broker 记录；
- 与 Service Worker 的消息经过身份和策略版本校验。

当前 native-compat 会由 Chromium 按原始 Manifest 注入 Content Script；如果某个平台无法在 Lite Runtime 中可靠阻止 Content Script 通过标准 Web API 绕过 Broker，安全模式必须关闭 Content Script，而不是把它标记为“已隔离”。

## 网络模型

网络能力分为两种运行时边界：

1. **Phosona 声明式网络规则**：只允许阻止等有限动作，不读取请求正文、Cookie 或完整敏感请求头。
2. **原始 WebExtension native-compat**：由 Chromium 执行其声明的 `webRequest`/`fetch` 等 API；它经过安装和权限门控，但不是逐调用 Rust 代理。严格 Lite Runtime 才允许通过 Broker Shim 提供这些操作。

在严格 Lite Runtime 中，无法转换为可验证 Shim 操作的 `webRequestBlocking` 回调默认拒绝。native-compat 保留原始回调语义，并在 UI 中明确显示非严格代理状态。

## 首期支持范围

### 当前支持

- Manifest V2/V3 原始解包扩展包；
- 原始 Manifest V2 background page 由 Chromium 扩展宿主执行；Manifest V3 包可以被识别，但当前 Electron 会对 Service Worker/未实现 API 显示兼容性提示，不能因此承诺 MV3 后台已经可用；
- uBlock Origin 的 Chromium 构建包可按原包方式加载（前提是当前 Electron/Chromium 支持其所需 API）；Firefox 构建包只有在其 Manifest/API 与当前宿主兼容时才可用；
- 必需权限默认拒绝；授权后才加载，停用或撤销后卸载；
- 包摘要绑定、完整权限披露、系统永久拒绝能力和加载状态审计。

### 后续支持

- Popup 和 Options 页面；
- 用户动作绑定的 `activeTab`；
- host scope 保护的 Content Script；
- 有限 `scripting` 注入；
- 动态和会话级声明式规则。

### 永久拒绝

- `process.execute`、Shell、Native Messaging；
- Cookies、凭据和 Token；
- Debugger、系统代理和整机网络接管；
- 任意文件路径访问；
- 桌面捕获和系统级输入监控；
- 任意 WASM、远程脚本和应用内部对象。

## uBlock 原始扩展

uBlock 不在 Phosona 内重新实现。安装时选择 uBlock 的单文件构建产物（例如 Chromium `.zip` / `.crx` 包），插件管理器在受控 staging 目录读取原始 `manifest.json`，验证后把 background 脚本、Service Worker、规则资源、Popup 和其他文件保存到应用的插件目录，并用包内容摘要作为插件身份的一部分：

```text
原始 uBlock 包 → Manifest/权限披露 → Rust 权限策略 → native-compat 扩展宿主 → 原始 uBlock 代码
```

当前 Electron 版本对扩展 API 的支持范围仍是实际限制：如果 uBlock 构建要求 Electron 未实现的 API，界面会显示加载失败原因，而不会静默改用一个“伪 uBlock”或规则翻译器。

## 验收标准

- 扩展代码无法读取 Node、Electron、preload 或应用内部对象。
- 伪造 Manifest、修改权限范围、重复提交 IPC 都不能获得额外权限。
- 扩展包变化后旧持久授权失效。
- `fetch`、WebSocket、下载、剪贴板和脚本注入请求都能关联扩展身份。
- 未授权 host、端口、tab、frame 和文件范围全部拒绝。
- 撤销后新请求立即失败，旧上下文停止或降权。
- Broker 退出时扩展特权能力 fail-closed。
- 审计不记录剪贴板内容、文件内容、Cookie、Token 和 URL 查询串。
- 普通网页、浏览器导航、WebSocket、下载和 aria2 不受扩展失败影响。
- Chrome 与 Firefox 适配器共享同一套 Rust 权限判定测试。

## 当前状态

当前项目已经具备安全声明式插件运行时、Rust Broker、摘要绑定、权限撤销、原始 WebExtension native-compat 宿主和实际效果审计。严格的 Rust-mediated Lite Runtime 仍是后续增强，不能把当前 Electron 原生适配器描述为已经完成：

```text
Manifest Compiler
  → 原始 WebExtension native-compat 宿主（当前）
  → Lite Runtime Supervisor（严格模式）
  → API Shim
  → Service Worker 生命周期
  → Popup / Content Script
  → Chrome / Firefox 兼容性测试
```

相关基础规范见 [`docs/plugin-architecture.md`](./plugin-architecture.md)。

## 官方参考

- [Electron Extensions API](https://www.electronjs.org/docs/latest/api/extensions-api)
- [Chrome Manifest V3](https://developer.chrome.com/docs/extensions/develop/migrate/what-is-mv3)
- [Chrome declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest)
- [Mozilla WebExtensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions)
