# WearEngine Bridge

An LSPosed/Vector module for developers working with Huawei Lite Wearable devices.

## What it solves

A custom-signed DevEco Assistant build or another WearEngine client may fail to
retrieve paired watches with **Scope unauthorized**, even when the watch is
connected to Huawei Health.

WearEngine Bridge runs inside Huawei Health and allows requests for the WearEngine
device-manager scope regardless of the client's package name or signing certificate.
It applies to all clients requesting that scope, not only DevEco Assistant.

The module does not install HAP files itself. Your client still handles transfer
and installation. Separate Health consent and watch-side installation checks remain
unchanged.

## Requirements and compatibility

- A rooted phone with LSPosed/Vector supporting legacy Xposed API 82.
- Android 8.0 (API 26) or newer. Device testing was performed on Android 15 with
  Magisk 31 and Vector/LSPosed; other Android versions are not confirmed.
- Huawei Health (`com.huawei.health`). The reference version is **17.0.8.300**
  (versionCode `1700008300`).

Other Health versions are not blocked: the module logs a warning and attempts to
hook the expected method. Compatibility with those versions is unconfirmed. If the
required class or method contract does not match, the hook cannot be applied.

The developer confirmed installation and operation of the signed **0.1** module on
their phone. HAP installation through a client has not been separately confirmed.

## Installation

1. Download a signed APK from [this repository's releases](https://github.com/Xposed-Modules-Repo/ru.randgor.wearenginebridge/releases)
   or the [source project's releases](https://github.com/RandGor/wearengine-bridge/releases).
2. Install the APK and enable **WearEngine Bridge** in Vector/LSPosed.
3. Select **Huawei Health (`com.huawei.health`)** as the module's scope.
4. Keep Huawei Health out of an enforced Magisk DenyList.
5. Force-stop Huawei Health through Android settings, then reopen Health and your
   client, such as DevEco Assistant. Check that the client can retrieve the watch.

Disable Frida scripts or other implementations of the same bypass while testing.
The module has no launcher or configuration screen and requests no additional
Android permissions.

After updating the module, restart Health to load the updated hooks. If they do not
load, reboot the phone. To undo the change, disable the module and restart Health;
clearing Health's data is unnecessary.

Updates must use the same signing key. If an earlier development build has a higher
versionCode, uninstall only that module before installing 0.1. Do not uninstall
Huawei Health.

## Scope of the change

The module hooks `ScopeManager.checkScopeAvailability` in Health's `:DaemonService`
and `:PhoneService` processes. It allows the combination
`scope = com.huawei.hiwear.devicemanager` and `channel = wearEngine` for any client.
Other scope/channel combinations retain their original behavior.

## Troubleshooting and support

Messages are written both to the Vector/LSPosed module log and to Android Logcat.
They are not shown as notifications or inside DevEco Assistant.

**On the phone:** open the Vector/LSPosed manager, open its **Logs** section
(the label may vary by version or language), and look for `WearEngineBridge`
in the module log. If there is no search field, export the log and search the
saved text.

**In Android Studio:** open **View > Tool Windows > Logcat**, select the connected
phone, and enter `tag:WearEngineBridge` in the filter field. Include **Info** level
messages. Remove `package:mine` or any filter restricted to the module or DevEco
Assistant: the messages come from Huawei Health's `:DaemonService` and
`:PhoneService` processes.

To capture a fresh hook installation, start viewing the log, force-stop Huawei
Health through Android settings, then reopen Health and the WearEngine client and
repeat the request. Messages are event-driven; there is no periodic heartbeat.
`READY`, `CLASS_INACTIVE` and `CLASS_HOOK_FAILED` are available in normal builds;
a diagnostic build is only needed for additional class-loader and per-call details.

Log tag: **WearEngineBridge**. Normal builds log hook readiness and errors;
diagnostic builds also log class loading and individual calls.

- `READY`: a target hook was installed; verify the client's actual result.
- `WARN untested Health`: the installed Health version differs from the reference
  version; hook installation is still attempted.
- `CLASS_INACTIVE` or `CLASS_HOOK_FAILED`: a target method could not be hooked.

Report problems in [the source project's Issues](https://github.com/RandGor/wearengine-bridge/issues).
Include your Health, Android, framework and module versions, reproduction steps,
and relevant logs with personal information removed.

## Source and license

[Source code](https://github.com/RandGor/wearengine-bridge) ·
[Changelog](https://github.com/RandGor/wearengine-bridge/blob/main/CHANGELOG.md) ·
[MIT license](https://github.com/RandGor/wearengine-bridge/blob/main/LICENSE)

This repository contains the module's catalog description and release APKs.
Development and builds are maintained in the source repository.
This is an independent project, not affiliated with Huawei. Huawei APKs are not
modified or distributed by this project.
