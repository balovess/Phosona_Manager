# Chrome 和 Firefox 扩展兼容性

本页说明在 Phosona Manager 中安装现有 Chrome 或 Firefox 扩展时，你可以期待什么效果。

[English](webextension-compatibility.md)

## 两种不同的插件体验

### Phosona 声明式插件

这类插件只包含规则和设置，不执行插件 JavaScript，适合用于请求过滤等功能。它们只能使用清单中声明并经你批准的权限，是更安全的选择。

### 原始 WebExtension 扩展

Phosona 可以尝试加载带有 Manifest V2 或 V3 `manifest.json` 的 Chrome 或 Firefox 原始扩展包。这类扩展运行在浏览器扩展环境中，并标记为“原生兼容”。扩展自己的后台页面、Service Worker、弹窗和内容脚本仍属于扩展本身。

“原生兼容”不等于完整兼容 Chrome 或 Firefox。Electron 只支持部分浏览器扩展 API，因此扩展可能显示兼容性提示、只能部分工作，或无法加载。

## 安装扩展

1. 在“设置 → 插件”中选择一个 `.zip`、`.xpi` 或 `.crx` 安装包。
2. 查看开发者、安装包指纹和所需权限。
3. 确认安装。
4. 只批准你理解且确实需要的权限。

应用会在安装前检查安装包，不会运行安装脚本。安装包内容发生变化后，需要重新确认权限。

## 常见兼容性限制

如果扩展依赖 Electron 未提供的浏览器 API、需要不支持的后台行为，或使用 Firefox 专属功能，扩展可能无法正常工作。网站权限仍需要在 Phosona Manager 中单独批准。

uBlock 及类似扩展必须以其原始兼容扩展包安装。Phosona Manager 不会把扩展过滤列表转换成自己的规则格式。安装兼容包也不代表所有过滤规则和浏览器 API 都能正常运行。

## 扩展没有加载时

插件页面可能显示扩展正在等待权限、与当前浏览器环境不兼容，或加载失败。你可以在“设置”中禁用或删除它。安装包或权限发生变化后，扩展会被卸载并需要重新审核。

## 当前状态

当前版本可以识别 Manifest V2 和 V3 原始扩展包；当所需 API 可用时，它们可能通过 Electron 原生扩展环境加载。当前版本不承诺完整的 Chrome/Firefox API 兼容性，也不承诺已经提供完全由权限代理控制的独立扩展运行时。

通用安装流程见[插件使用指南](../plugins/README_CN.md)，权限说明见[插件权限说明](plugin-architecture_CN.md)。
