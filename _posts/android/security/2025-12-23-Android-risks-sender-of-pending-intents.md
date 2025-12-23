---
title: Android-安全风险-
date: 2025-12-23 10:58:04 +0800
categories:
  - Android
  - Security
  - Risks
  - OWASP
tags:
  - Android
  - Security
  - Risks
  - OWASP
description: 信任 PendingIntent 的发送者，这可能会导致安全漏洞。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

使用 [`PendingIntent.getCreator*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 或 [`PendingIntent.getTarget*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 来确定是否信任 PendingIntent 的发送者会带来被攻击的风险。

[`PendingIntent.getCreator*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 或 [`PendingIntent.getTarget*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 会返回 PendingIntent 的创建者，但该创建者并不总是与其发送者一致。创建者可能值得信任，但发送者**绝不**值得信任，因为发送者可能是恶意应用，它通过各种机制获取了另一个应用的 PendingIntent，例如：

- 来自：[`NotificationListenerService`](https://developer.android.com/reference/android/service/notification/NotificationListenerService?hl=zh-cn)
- 属于存在漏洞的应用程序的合法使用案例。

[`PendingIntent.getCreator*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 或 [`PendingIntent.getTarget*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 的合法用法示例是显示 PendingIntent 将要启动的应用程序的图标。

## 影响

仅仅因为你查询过（并且信任）创建者就信任 PendingIntent 的发送者，这可能会导致安全漏洞。如果一个应用仅仅因为创建者就信任 PendingIntent 的发送者，然后共享其身份验证或授权逻辑，这样，每当 PendingIntent 的发送者是恶意应用程序时，根据易受攻击的应用程序代码的实现方式，这可能会导致身份验证绕过，甚至可能基于无效的、不受信任的输入执行远程代码。

## 缓解措施

### 区分发送者和创作者

接收 PendingIntent 时执行的任何类型的身份验证或授权逻辑都不得基于以下假设：PendingIntent 的创建者是使用 [`PendingIntent.getCreator*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 或 [`PendingIntent.getTarget*()`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#public-methods_1) 标识的。

### 使用其他方式验证调用方

如果需要验证调用者身份，则应使用 Service 或 ContentProvider，而不是 PendingIntent——两者都允许使用 [Binder.getCallingUid()](https://developer.android.com/reference/android/os/Binder?hl=zh-cn#getCallingUid\(\)) 获取调用方 UID。可以使用 [PackageManager.getPackagesForUid()](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#getPackagesForUid\(int\)) 查询 UID。

从 API 级别 34 开始，还可以使用[BroadcastReceiver.getSentFromUid()](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=zh-cn#getSentFromUid\(\)) 或 [BroadcastReceiver.getSentFromPackage()](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=zh-cn#getSentFromPackage\(\)) 前提是发送方已选择在广播期间使用 [BroadcastOptions.isShareIdentityEnabled()](https://developer.android.com/reference/android/app/BroadcastOptions?hl=zh-cn#isShareIdentityEnabled\(\)) 共享身份。

您应该始终检查调用包是否具有预期的签名，因为侧载包的包名可能与 Play 商店中的包名重叠。

## 资源

- [Pending intent](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn)

