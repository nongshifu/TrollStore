# 巨魔商店

TrollStore 是一个永久签名的监禁应用程序，可以永久安装您在其中打开的任何 IPA。

它之所以有效，是因为AMFI/CoreTrust错误，iOS不会验证用于签署二进制文件的根证书是否合法。

## 安装巨魔商店

### 安装指南

| 版本/设备 |arm64处理器 （A8 - A11） |arm64e处理器 （A12 - A15， M1） |
| --- | --- | --- |
| 13.7 及以下 | 不支持（CT 错误仅在 14.0 中引入） | 不支持（CT 错误仅在 14.0 中引入） |
| 14.0 - 14.8.1 | [checkra1n + TrollHelper](./install_trollhelper.md) | [TrollHelperOTA (arm64e)](./install_trollhelperota_arm64e.md) |
| 15.0 - 15.4.1 | [TrollHelperOTA (iOS 15+)](./install_trollhelperota_ios15.md) | [TrollHelperOTA (iOS 15+)](./install_trollhelperota_ios15.md) |
| 15.5 beta 1 - 4 | [TrollHelperOTA (iOS 15+)](./install_trollhelperota_ios15.md) | [TrollHelperOTA (iOS 15+)](./install_trollhelperota_ios15.md) |
| 15.5 (RC) | 不支持（CT 错误已修复） | 不支持（CT 错误已修复） |
| 15.6 beta 1 - 5 | [SSH Ramdisk](./install_sshrd.md) | [TrollHelperOTA (arm64e)](./install_trollhelperota_arm64e.md) |
| 15.6 (RC1/2) 及以上 | 不支持（CT 错误已修复） | Not Supported (CT Bug fixed) |

此版本表是最终版本，TrollStore将永远不支持此处列出的版本以外的任何其他版本。不要问，如果你的设备版本不受支持，最好忘记TrollStore的存在。

## 正在更新TrollStore

当新的TrollStore更新可用时，TrollStore设置的顶部将显示一个安装按钮。点击按钮后，TrollStore将自动下载更新、安装并重新发布。

或者（如果出现任何问题），您可以在Release下下载TrollStore.tar文件，并在TrollStore中打开它，TrollStore将安装更新并重新发布。

## 卸载应用

从TrollStore安装的应用程序只能从TrollStore本身卸载，点击应用程序或在“应用程序”选项卡中将其向右滑动以将其删除。

## 持久性帮助程序 Persistence Helper

TrollStore中使用的CoreTrust漏洞只足以安装“系统”应用程序，这是因为FrontBoard在每次启动用户应用程序之前都会进行额外的安全检查（它调用libmis）。不幸的是，无法安装通过图标缓存重新加载而保留的新“系统”应用程序。因此，当iOS重新加载图标缓存时，所有安装了TrollStore的应用程序，包括TrollStore本身，都将恢复到“用户”状态，不再启动。

解决这一问题的唯一方法是在系统应用程序中安装一个持久性帮助程序，然后可以使用该帮助程序将TrollStore及其安装的应用程序重新注册为“系统”，以便它们能够再次启动，TrollStore设置中提供了一个选项。

在越狱的iOS 14上，当使用TrollHelper进行安装时，它位于/Applications中，并将通过图标缓存重新加载作为“系统”应用程序进行持久化，因此TrollHelpers被用作iOS 14上的持久化助手。

## URL Scheme

从1.3版本开始，TrollStore取代了系统URL方案“apple amplifier”（这样做是为了让“越狱”检测无法像TrollStore有唯一URL方案那样检测到TrollStore）。此URL方案可用于直接从浏览器安装应用程序，格式如下：

`apple-magnifier://install?url=<URL_to_IPA>`

在没有安装TrollStore（1.3+）的设备上，这只会打开放大镜应用程序。

## Features

IPA中的二进制文件可以具有任意权限，使用ldid和您想要的权限对其进行伪造签名 (`ldid -S<path/to/entitlements.plist> <path/to/binary>`) TrollStore将在安装时使用伪造的根证书退出时保留这些权利。这给了你很多可能性，下面将对其中一些可能性进行解释。

### Banned entitlements

A12+上的iOS 15已经禁止了以下三项与运行未签名代码有关的权利，如果没有PPL绕过，这些权利是不可能获得的，使用它们签名的应用程序将在发布时崩溃。

`com.apple.private.cs.debugger`

`dynamic-codesigning`

`com.apple.private.skip-library-validation`

### Unsandboxing

您的应用程序可以使用以下权利之一在不受限制的情况下运行：

```xml
<key>com.apple.private.security.container-required</key>
<false/>
```

```xml
<key>com.apple.private.security.no-container</key>
<true/>
```

```xml
<key>com.apple.private.security.no-sandbox</key>
<true/>
```

如果您仍然需要应用程序的沙盒容器，则建议使用第三种方法。

您可能还需要平台应用程序权限才能正常工作：

```xml
<key>platform-application</key>
<true/>
```

请注意，平台应用程序权限会导致副作用，例如沙盒的某些部分变得更紧，因此您可能需要额外的私人权限来规避这一点。（例如，之后您需要为要访问的每个IOKit用户客户端类提供一个异常权限）。

为了使用 `com.apple.private.security.no-sandbox` and `platform-application` t为了能够访问它自己的数据容器，您可能需要额外的权限：

```xml
<key>com.apple.private.security.storage.AppDataContainers</key>
<true/>
```

### Root Helpers

当你的应用程序没有沙盒时，你可以使用posix_spown生成其他二进制文件，你也可以使用以下权限以root身份生成二进制文件：entitlement:

```xml
<key>com.apple.private.persona-mgmt</key>
<true/>
```

您还可以将自己的二进制文件添加到应用程序捆绑包中。

之后，您可以使用 [TSUtil.m中的spawnRoot函数](./Shared/TSUtil.m#L74) 生成二进制文件作为根。

### 使用TrollStore无法实现的事情

- 获得适当的平台化 (`TF_PLATFORM` / `CS_PLATFORMIZED`)
- 生成启动后台程序 (Would need `CS_PLATFORMIZED`)
- 将代码注入系统进程 (Would need `TF_PLATFORM`, a userland PAC bypass and a PMAP trust level bypass)

## Credits and Further Reading

[@LinusHenze](https://twitter.com/LinusHenze/) - Found the CoreTrust bug that allows TrollStore to work.

[Fugu15 Presentation](https://youtu.be/NIyKNjNNB5Q?t=3046)

[Write-Up on the CoreTrust bug with more information](https://worthdoingbadly.com/coretrust/).
