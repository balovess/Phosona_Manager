# Phosona Manager Plugin Guide

Plugins add optional features to Phosona Manager. You can use them to filter page requests, find browser resources, or connect a browser extension to the download manager.

[简体中文](README_CN.md)

## Install a plugin

1. Open **Settings → Plugins**.
2. Select **Add plugin**.
3. Choose a single `.zip`, `.xpi`, or `.crx` package.
4. Review the developer, package fingerprint, and requested permissions.
5. Confirm the installation. The plugin remains disabled and its permissions remain off until you enable them.

Do not install a package from an unknown source. Only install a plugin when you understand why it needs each requested permission.

## Permission choices

Each permission is shown separately. Depending on the capability, you may choose **Allow once**, **Allow for this session**, **Always allow**, or **Deny**.

Examples include:

- browser access for a declared website;
- creating a download after your approval;
- reading or writing the clipboard after a user action;
- reading or writing a specific folder;
- showing notifications or updating the system tray.

Permissions are denied by default. You can revoke them in Settings at any time. A changed package is treated as a new version and requires permission approval again.

## Two plugin types

Phosona declarative plugins contain rules and settings only. They do not run plugin JavaScript and are the safest choice for request filtering.

Original Chrome and Firefox extensions can also be loaded when their Manifest V2 or V3 package is compatible with the built-in browser. They run in the browser extension environment and are labelled **native compatibility**. Not every Chrome or Firefox API is available in Electron, so an extension may work partly or fail to load.

## Troubleshooting

- If a plugin does not load, check that the package is valid and that all required permissions have been approved.
- If the plugin asks for a permission you do not want to grant, leave that permission denied; unrelated features can remain available.
- If a package has changed, review and grant its permissions again.
- If the download engine or plugin security service is unavailable, plugin capabilities are temporarily disabled. Normal downloads and browsing can continue.

See [Plugin permissions](../docs/plugin-architecture.md) and [Browser extension compatibility](../docs/webextension-compatibility.md) for more detail.
