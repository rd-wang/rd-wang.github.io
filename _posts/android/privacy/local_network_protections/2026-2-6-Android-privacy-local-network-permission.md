---
title: Android-隐私权-本地网络权限
date: 2026-2-6 15:26:30 +0800
categories:
  - Android
  - Privacy
  - Local_network_protections
tags:
  - Android
  - Privacy
  - Local_network_protections
description: 具有 INTERNET 权限的任何应用都可以访问局域网 (LAN) 中的设备。这使得应用可以轻松连接到本地设备，但也带来了隐私方面的隐患，例如形成用户指纹和成为位置信息代理。
math: true
---
任何拥有`INTERNET` 权限的应用程序都可以访问局域网 (LAN) 上的设备。这使得应用程序能够轻松连接到本地设备，但也带来了隐私问题，例如形成用户的指纹并充当位置代理。

本地网络保护项目旨在通过新的运行时权限来限制对本地网络的访问，从而保护用户的隐私。

## 影响

在 Android 16 期间，此权限是一项选择启用功能，这意味着只有选择启用的应用会受到影响。选择启用该功能的目标是让应用开发者了解应用的哪些部分依赖于隐式本地网络访问权限，以便他们为在未来的 Android 版本中对这些部分进行权限保护做好准备。

如果应用使用以下方式访问用户的本地网络，则会受到影响：

- 直接或通过库使用本地网络地址（例如 `Multicast DNS (mDNS)` 或 `Simple Service Discovery Protocol (SSDP)`）上的原始套接字。
- 使用可访问本地网络的框架级类，例如 `NsdManager`。

### 影响详情

与本地网络地址之间的流量需要本地网络访问权限。下表列出了一些常见情况：

|应用低级层网络操作|需要本地网络权限|
|---|---|
|建立出站 TCP 连接|是|
|接受传入的 TCP 连接|是|
|发送 UDP 单播、多播、广播|是|
|接收传入的 UDP 单播、多播、广播|是|

这些限制是在网络堆栈深处实现的，因此适用于**所有网络 API**。这包括在平台或受管理的代码中创建的套接字、Cronet 和 OkHttp 等网络库，以及基于这些库实现的任何 API。尝试解析具有 `.local` 后缀的本地网络上的服务需要本地网络权限。

**注意**： 需要本地网络访问权限的 Android WebView 发起的流量将继承宿主应用的权限状态

上述规则的例外情况：

- 如果设备的 DNS 服务器位于本地网络上，则进出该服务器（在端口 53 上）的流量不需要本地网络访问权限。
- 使用 Output Switcher 作为其应用内选择器的应用程序不需要本地网络权限（更多指南将在以后的版本中提供）。

**注意**：许多媒体投屏场景依赖于本地网络访问，因此会受到此次变更的影响。但是，并非所有提供投屏功能的应用都需要申请新的权限。未来将在 Android 的后续版本中提供有关处理投屏场景的 API 和指南。

## 指南

如需选择启用本地网络限制，请执行以下操作：

1. 将设备刷写到搭载 Android 16 Beta 3 或更高版本的 build
2. 安装要测试的应用
3. 使用 adb 切换 Appcompat 配置
    
    ```bash
    adb shell am compat enable RESTRICT_LOCAL_NETWORK <package_name>
    ```
    
4. **重新启动设备**
    

现在，您的应用对本地网络的访问权限受到限制，任何访问本地网络的尝试都会导致套接字错误。如果您使用的是在应用进程之外执行本地网络操作的 API（例如 `NsdManager`），则在选择启用该功能期间，这些 API 不会受到影响。

如需恢复访问权限，您必须向应用授予 `NEARBY_WIFI_DEVICES` 权限。

- 确保应用在其 `manifest` 中声明了 `NEARBY_WIFI_DEVICES` 权限。
- 依次前往**设置** > **应用** > **[应用名称]** > **权限** > **附近的设备** > **允许**

**注意**： 在未来的 Android 版本中，此功能将受到 [`NEARBY_DEVICES`](https://developer.android.com/reference/android/Manifest.permission_group?hl=zh-cn#NEARBY_DEVICES) 权限组中新权限的保护。

现在，应用对本地网络的访问权限应该已恢复，并且所有场景都应像选择启用应用之前一样正常运行。应用网络流量将受到以下影响。

| 权限  | 出站 LAN 请求 | 出站/入站互联网请求 | 入站 LAN 请求 |
| --- | --------- | ---------- | --------- |
| 已授予 | Works     | Works      | Works     |
| 未授予 | Fails     | Works      | Fails     |

使用以下命令切换关闭 Appcompat 配置

```bash
adb shell am compat disable RESTRICT_LOCAL_NETWORK <package_name>
```

### 错误

每当调用套接字调用 `send` 或 `send` 变体来访问本地网络地址时，系统都会向该套接字返回因这些限制而产生的错误。

错误示例：

```bash
sendto failed: EPERM (Operation not permitted)

sendto failed: ECONNABORTED (Operation not permitted)
```

### Bugs

[提交以下方面的 bug](https://developer.android.com/about/versions/16/feedback?hl=zh-cn) 和反馈：

- LAN 访问权限存在差异（您认为某些访问权限不应被视为“本地网络”访问权限）
- 应禁止但未禁止 LAN 访问的 bug
- 本应不被阻止但被阻止的 LAN 访问权限的 bug

以下内容应不会受到此变更的影响：

- 访问互联网
- 移动网络