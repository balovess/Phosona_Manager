# Chrome and Firefox Extension Compatibility

This page explains what to expect when installing an existing Chrome or Firefox extension in Phosona Manager.

[简体中文](webextension-compatibility_CN.md)

## Two different plugin experiences

### Phosona declarative plugins

These plugins contain rules and settings rather than plugin JavaScript. They are the safer option for request filtering and similar tasks. They can only do what their declared permissions allow.

### Original WebExtensions

Phosona can load some original Chrome or Firefox packages with a Manifest V2 or V3 `manifest.json`. These extensions run in the browser's extension environment and are labelled **native compatibility**. Their own background pages, service workers, popups, and content scripts remain part of the extension.

Native compatibility is not the same as full Chrome or Firefox compatibility. Electron supports only part of the browser extension API, so an extension may show compatibility warnings, work partially, or fail to load.

## Install an extension

1. Choose a single `.zip`, `.xpi`, or `.crx` package from **Settings → Plugins**.
2. Review the package developer, fingerprint, and requested permissions.
3. Confirm the installation.
4. Approve only the permissions you understand and need.

The application checks the package before installation and does not run installation scripts. A changed package requires permission approval again.

## Common compatibility limits

An extension may not work when it depends on browser APIs that Electron does not provide, requires unsupported background behavior, or expects Firefox-only features. Website permissions still need to be approved in Phosona Manager.

uBlock and similar extensions must be installed as their original compatible extension package. Phosona Manager does not convert an extension's filter lists into its own rule format. Installing a compatible package does not guarantee that every filter or browser API will work.

## When an extension is not loaded

The plugin page may show that an extension is waiting for permission, is incompatible with the current browser host, or failed to load. You can disable or remove it from Settings. If the package or its permissions change, the extension is unloaded and must be reviewed again.

## Current status

Original Manifest V2 and V3 packages can be recognized and may load through Electron's native extension host when their required APIs are available. Full Chrome/Firefox API compatibility and a separate fully brokered extension runtime are not promised by the current version.

For the general installation flow, see the [Plugin Guide](../plugins/README.md). For permission details, see [How plugin permissions work](plugin-architecture.md).
