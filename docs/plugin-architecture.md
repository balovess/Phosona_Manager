# How Plugin Permissions Work

This page explains the plugin permission model in user-facing terms.

[简体中文](plugin-architecture_CN.md)

## Plugins start with no access

Installing a plugin does not give it access to your browser, files, clipboard, downloads, or network requests. Every permission starts as **Deny**. The plugin can be enabled without granting every permission it requests.

Each permission has a clear purpose and, where applicable, a website or folder scope. Allowing access to one website does not allow access to other websites. Allowing one folder does not allow access to the rest of your device.

## Permission durations

- **Allow once** ends after the current action.
- **Session** ends when the application session ends.
- **Always allow** remains available until you revoke it or the package changes.
- **Deny** prevents the capability from being used.

Revoking one permission does not disable unrelated permissions. If the plugin package changes, existing permanent approvals are cleared so that you can review the new package again.

## What plugins can and cannot do

Depending on the permissions you approve, a plugin may be able to observe declared browser requests, create an approved download, show a notification, use the clipboard after a user action, or access a declared folder.

Plugins cannot use arbitrary Node.js or Electron APIs, run shell commands, open arbitrary local files, or silently expand a website or folder scope. Process and shell execution are not available to plugins in the current version.

## Privacy and audit information

The plugin page shows the permissions you granted and a redacted record of important decisions, such as a blocked request or a denied folder access. Audit information does not include clipboard contents, file contents, Cookies, tokens, or URL query strings.

If the plugin security service is unavailable, plugin capabilities fail closed. This means the plugin loses access temporarily instead of receiving broader access. Normal browser and download features are not disabled by this failure.

## Updating and removing plugins

Plugin updates must keep the same plugin identity and increase its version. New permissions or a broader website scope require a new approval. You can disable or remove a plugin from Settings at any time.

For installation steps, see the [Plugin Guide](../plugins/README.md). For Chrome and Firefox extension behavior, see [Browser extension compatibility](webextension-compatibility.md).
