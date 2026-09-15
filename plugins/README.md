# Phosona Manager plugins

See [`docs/plugin-architecture.md`](../docs/plugin-architecture.md) for storage locations, manifest rules, permission levels, and review requirements.

Chrome/Firefox WebExtension 的兼容边界见 [`docs/webextension-compatibility.md`](../docs/webextension-compatibility.md)。用户通过“添加插件”选择单文件 `.zip`、`.xpi` 或 `.crx` 包；系统安全解压后才保存到应用管理的 `userData/plugins/<plugin-id>/`。运行时可以加载原始扩展，但这条路径会明确标记为 Electron 原生兼容适配，不把它伪装成 Rust 严格代理运行时。

Built-in plugins are discovered from this directory in a packaged build. User packages are discovered from the application user-data `plugins/` directory at runtime after the single-file installation flow has extracted them. The managed runtime layout is one directory per validated plugin, containing a `manifest.json`:

```json
{
  "id": "example.plugin",
  "name": "Example plugin",
  "version": "1.0.0",
  "description": "Short description",
  "developer": { "name": "Example Labs", "contact": "security@example.invalid" },
  "permissions": [
    { "id": "browser.webRequest", "scope": "https://example.com", "purpose": "Inspect declared origin requests" },
    { "id": "downloads.create", "purpose": "Create user-approved downloads" }
  ],
  "runtime": {
    "type": "declarative",
    "requestRules": [
      {
        "id": "block-pixel",
        "action": "block",
        "urlPattern": "https://example.com/pixel*",
        "resourceTypes": ["script", "image", "ping", "fetch", "xhr"]
      }
    ]
  }
}
```

The supported Phosona-native runtime is declarative network filtering. A rule can only block requests, must use an exact `http(s)` origin already declared by a granted `browser.webRequest` permission, and must list explicit non-main-frame resource types. The browser request seam evaluates enabled rules in the Electron session, while the Rust `plugin-broker` remains the authority for plugin identity, scope, grant mode, and audit. This native format is separate from WebExtension packages and is not a uBlock compatibility layer.

An original Chrome/Firefox package is also accepted when its root `manifest.json` contains `manifest_version` 2 or 3. It is copied byte-for-byte into the user plugin directory, identified by the package digest, and loaded by the browser session only after every required, non-system-denied capability has been granted. Its original background page or Manifest V3 Service Worker remains the extension's code; Phosona does not rewrite its filter engine. Stop, revoke, or package-digest changes unload the native extension immediately. This path is shown as `native-compat`, because Electron exposes only a subset of browser extension APIs and does not provide a hook that can make every native extension API call pass through the Rust broker. It is therefore not the strict brokered runtime.

The desktop process validates manifests, shows the complete permission disclosure and package digest before installation, installs user plugins under the Electron `userData/plugins` directory, and persists only persistent grants. Every permission starts denied. Grants are independent and can be `once`, `session`, `persistent`, or `denied`; persistent grants are bound to the package SHA-256 digest, and changing package contents clears them. The application permission prompt and settings page can revoke grants immediately. Native WebExtensions require session/persistent grants for their required permissions because a long-lived extension cannot be safely represented by an allow-once lifetime. `process.execute` is permanently denied in v1. If the Rust broker is unavailable, all plugin capabilities are denied while normal browser and aria2 features continue to work. Untrusted plugin code is never evaluated in the renderer or main process. Chrome/Firefox extension scripts, DOM content scripts, arbitrary redirects, and arbitrary Node/Electron access are only available inside the explicitly labelled native-compat extension host.

The plugin page exposes a redacted “actual effects and permission audit” view. It can show that a declared pixel rule was blocked, that clipboard access was denied without a user action, that a filesystem path escaped its exact scope, or that external process execution was permanently denied. For an original WebExtension it also shows whether the unmodified package is loaded, the Manifest version, and why it is waiting for permissions or failed to load. Audit entries never include clipboard/file contents, cookies, tokens, or URL query strings.
