# Phosona Manager

Phosona Manager 是一款跨平台桌面下载管理器。它使用 Electron 提供桌面窗口和浏览器工作区，使用 Preact 构建界面，并管理内置的 Rust 下载引擎。

本仓库只用于发布 Phosona Manager 的二进制安装包；安装包和更新所需的元数据请在 [GitHub Releases](https://github.com/balovess/Phosona_Manager/releases) 中获取。

[English](README.md)

[隐私协议](PRIVACY.md)

## 界面截图

> 以下界面截图仅供展示和参考，不代表正式版最终界面；正式版本的界面可能会有较大差异。

### 下载控制中心

![下载控制中心](01.png)

### 语言选择

![语言选择](02.png)

## 主要功能

- HTTP、HTTPS 和 BitTorrent 下载
- URL、磁力链接和 `.torrent` 文件添加
- 暂停、继续、删除、队列管理和下载目录配置
- 并发数、连接数、速度限制、代理和其他 aria2 传输选项
- 内置浏览器工作区、多标签页、历史记录和书签
- 浏览器资源观察器：发现页面中的媒体、文件和流媒体资源，并将选定资源加入下载队列
- 浏览器下载统一交给下载管理器处理，可按设置转发必要的请求头和 Cookie
- 下载悬浮窗、系统托盘、开机启动和始终置顶
- BitTorrent Peers、Tracker、DHT 和做种状态查看
- 隐私保护模式、权限默认拒绝和浏览数据清理
- 权限约束的声明式插件系统
- 多语言界面，并支持阿拉伯语 RTL 布局

## 下载与安装

在 [Releases](https://github.com/balovess/Phosona_Manager/releases) 下载与你的系统和 CPU 架构对应的安装包：

| 系统 | 安装包 |
| --- | --- |
| Windows | `Phosona-Manager-<version>-win-<arch>.exe` |
| macOS | `Phosona-Manager-<version>-mac-<arch>.dmg` |
| Linux | `Phosona-Manager-<version>-linux-<arch>.AppImage` |

首次启动后，在“设置”中选择下载目录、并发数、代理、BitTorrent 和隐私选项。应用会自动启动本地下载引擎，RPC 仅监听本机回环地址，不对局域网开放。

## 隐私与安全边界

- 下载引擎由应用管理，RPC 端口和随机密钥不会暴露到界面或日志。
- 浏览器页面运行在受限的 Electron 会话中，不获得 Node.js 或 Electron 主进程权限。
- 浏览器权限默认拒绝；Cookie 转发、资源观察和插件权限需要用户明确开启。
- 插件当前只支持声明式网络规则，不执行插件 JavaScript，也不能直接访问 Node.js、Electron、Shell 或任意文件路径。
- 严格隐私模式会阻止已知的跟踪和指纹相关请求；这不是匿名网络或 VPN。

## 当前版本说明

当前版本为 `0.1.0 Preview`。部分能力仍在持续验证中，正式发布前请在干净环境中测试安装、下载恢复、BitTorrent、浏览器资源捕获和卸载流程。

应用内自动更新需要发布环境提供 HTTPS 更新源；如果当前版本显示“尚未配置更新源”，请通过 Releases 手动下载安装新版本。

## 项目关系

Phosona Manager 将下载任务交给 Rust 下载引擎执行，Electron 负责桌面窗口、浏览器会话和系统集成，前端负责交互界面。二进制发布不代表应用包含或替代所有上游项目的许可证和版权声明；使用前请阅读对应 Release 中的说明及随包提供的许可文件。
