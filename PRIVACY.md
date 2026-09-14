# Phosona Manager Privacy Policy

**Effective date:** 2026-09-14
**Last updated:** 2026-09-14

## 1. Maintainer

Phosona Manager is maintained by the project maintainer `balovess`.

Project page: <https://github.com/balovess/Phosona_Manager>

## 2. Data collection principles

The current version of Phosona Manager:

- does not require an account;
- does not provide a user account system;
- does not collect names, email addresses, phone numbers, or precise location;
- does not collect browsing history, download history, or downloaded files;
- does not collect Cookie contents;
- does not collect advertising identifiers;
- does not perform behavioral analytics, advertising profiling, or telemetry;
- does not provide an interface for uploading user data;
- does not send user information to the project maintainer; and
- does not sell, rent, or share user information with third parties.

Download tasks, browsing records, bookmarks, Cookies, cache, settings, and plugin configuration are processed locally on the user's device.

## 3. Local data

The application may store download tasks and queues, download directories, proxy and transfer settings, browser tabs, history and bookmarks, Cookies, cache, plugin configuration, permission state, and application preferences locally on the user's device.

This data is not automatically uploaded to the project maintainer. Users can delete local data through the application settings, the operating system file manager, or by uninstalling the application.

## 4. Network features initiated by the user

When a user actively downloads a file, visits a website, or uses a magnet link, Torrent, proxy, Tracker, or DHT, the application connects to the corresponding network services as requested by the user.

These connections are necessary for the download or browsing function itself. They are not Phosona Manager collecting or uploading user data to the project maintainer. Websites, download servers, Trackers, DHT nodes, and other network services may process requests under their own privacy policies. Phosona Manager does not operate or control those third-party services.

If a user enables Cookie, header, or resource forwarding, the relevant information is used only to complete the browsing or download request selected by the user.

## 5. Update checks

Apart from downloads and browsing initiated by the user, the application may make HTTPS requests to check for a new version.

Update checks are used only to determine whether a new version exists, match the operating system and CPU architecture, and provide applicable installer or update metadata. Update requests do not include download history, browsing history, Cookies, downloaded files, page content, plugin data, or user identity information.

As part of normal HTTPS communications, an update server may receive technical network information such as an IP address, request time, and client connection information. The project does not use this information to build user profiles or associate it with the user's download or browsing activity.

## 6. Browser and plugin security boundaries

Browser pages run in a restricted Electron session without Node.js, shell, or Electron main-process access.

The plugin system supports only permission-constrained declarative network rules. Plugins cannot execute arbitrary JavaScript, access Node.js or the Electron main process, execute shell commands, read arbitrary local files, bypass application permission policies, or upload user data to the project maintainer.

Browser permissions, Cookie forwarding, resource observation, and plugin permissions are disabled or denied by default and require explicit user action.

## 7. Data sharing

Phosona Manager does not provide users' personal information to third parties and does not sell or rent user data.

Apart from update checks and network requests initiated by the user for downloads or browsing, the application has no interface for sending user data to the project maintainer or third parties.

## 8. Security

The application uses the following security boundaries:

- The download-engine RPC listens only on the local loopback address.
- The RPC secret is not shown in the UI or ordinary logs.
- Browser pages do not receive main-process access.
- Permissions are denied by default.
- Plugin capabilities are constrained by declarative permission rules.
- Strict privacy mode blocks known tracking and fingerprinting requests.

Users remain responsible for protecting their device accounts, Cookies, proxy credentials, and private download links.

## 9. User rights

Because Phosona Manager primarily operates locally, users can directly delete local application data, browsing data, download tasks, Cookies, cache, and plugin configuration.

The project maintainer does not hold copies of users' download history, browsing history, Cookies, or downloaded files. Therefore, there is generally no user data held by the project maintainer to export or delete.

If a user believes that the application violates this policy, they may contact the maintainer through the project GitHub page: <https://github.com/balovess/Phosona_Manager>. Do not post passwords, Cookies, private download links, or other sensitive information in public Issues.

## 10. Children

Phosona Manager is not directed at children and does not knowingly collect children's personal information.

## 11. Changes to this policy

If the project adds accounts, cloud synchronization, telemetry, crash reporting, advertising, or other data processing, this policy will be updated before those features are enabled and will describe the relevant processing.

## 12. Current version

This policy applies to Phosona Manager `0.1.0 Preview`. Apart from HTTPS update checks and network requests initiated by the user for downloads or browsing, the current version provides no other project-initiated backend communication or user-data upload.

This policy describes the privacy behavior of the current Preview version. Before a formal commercial release, it should be reviewed against the actual operating entity and applicable laws.
