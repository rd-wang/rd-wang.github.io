---
title: Android-安全风险-粘性广播
date: 2025-12-23 16:00:45 +0800
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
description: 具体影响取决于粘性广播的使用方式以及传递给广播接收器的数据。一般来说，使用粘性广播可能会导致敏感数据泄露、数据篡改、未经授权访问以在其他应用程序中执行行为以及拒绝服务。
math: true
---
**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

Android 应用和 Android 系统可以使用广播作为消息传递系统，通知其他应用它们可能感兴趣的事件。粘性广播是一种特殊类型的广播，被发送的 intent 对象在广播完成后仍会保留在缓存中。系统可能会将粘性意图重新广播给后续注册的接收者。遗憾的是，粘性广播 API 存在一些安全方面的缺陷，因此在 Android 5.0（API 级别 21）中已被弃用。

### 人人都能访问粘性广播

粘性广播无法限制接收者只能拥有特定权限。因此，它们不适合广播敏感信息。您或许会想，在广播 `Intent` 中指定[应用软件包名称](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#setPackage\(java.lang.String\))，就能限制 `BroadcastReceivers` 的集合：

[Kotlin](https://developer.android.com/privacy-and-security/risks/sticky-broadcast?hl=zh-cn#kotlin)
```kotlin
val intent = Intent("com.example.NOTIFY").apply {
    setPackage("com.example.myapp")
}
applicationContext.sendBroadcast(intent)
```
[Java](https://developer.android.com/privacy-and-security/risks/sticky-broadcast?hl=zh-cn#java)

```java
Intent intent = new Intent("com.example.NOTIFY");
intent.setPackage("com.example.myapp");
getApplicationContext().sendBroadcast(intent);
```

在本例中，只有  `com.example.myapp` 包中的接收器才能在广播发送时接收到 Intent。但是，当 Intent 从粘性缓存重新广播时，包名过滤器不会生效。

在本例中，只有 软件包中的接收器能在广播发送时收到 intent。但是，重新广播粘性缓存中的 intent 时，系统无法应用软件包名称过滤条件。使用 [`registerReceiver()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#registerReceiver\(android.content.BroadcastReceiver,%20android.content.IntentFilter\)) 方法注册接收器时，粘性缓存中与指定过滤器匹配的所有意图都会重新广播到接收器，而不管接收器位于哪个包名中。

### 人人都能发送粘性广播

如需发送粘性广播，应用只需拥有 `android.permission.BROADCAST_STICKY` 权限即可，系统会在安装应用时自动授予该权限。因此，攻击者可以将任意 intent 发送给任一接收器，进而可能在未经授权的情况下访问其他应用。广播接收器可以将发送方限制为拥有特定权限的应用。但是，这样一来，接收器便无法接收来自粘性缓存的广播，因为系统不会在任何应用身份认证的上下文中发送这些广播，也不会针对具有任何权限的应用进行广播。

### 人人都能修改粘性广播

如果某个 intent 属于粘性广播，则该 intent 会取代之前在粘性缓存中具有相同操作、数据、类型、标识符、类和类别的任何实例。因此，攻击者很容易就能覆盖合法应用中粘性 intent 内的 extra 数据，而后可能会向其他接收器重新发送广播。

[`sendStickyOrderedBroadcast()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#sendStickyOrderedBroadcast\(android.content.Intent,%20android.content.BroadcastReceiver,%20android.os.Handler,%20int,%20java.lang.String,%20android.os.Bundle\)) 方法一次向一个接收器发送广播，以便优先级较高的接收器可以在广播传递给优先级较低的接收器之前先消费该广播。当接收器逐个顺序执行时，接收器可以向下传递结果（例如通过调用 [`setResultData()`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=zh-cn#setResultData\(java.lang.String\))），也可以[中止广播](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=zh-cn#abortBroadcast\(\))，阻止后续接收器接收广播。攻击者如果能够从合法应用程序接收粘性有序广播，就可以创建一个高优先级接收器来篡改广播结果数据或完全丢弃广播。

## 影响

具体影响取决于粘性广播的使用方式以及传递给广播接收器的数据。一般来说，使用粘性广播可能会导致敏感数据泄露、数据篡改、未经授权访问以在其他应用程序中执行行为以及拒绝服务。

## 缓解措施

请勿使用粘性广播。推荐的做法是使用非粘性广播，并结合其他机制（例如本地数据库）随时获取当前值。

开发者可以通过[权限](https://developer.android.com/guide/components/broadcasts?hl=zh-cn#restrict-broadcasts-permissions)或通过在 intent 上设置[应用软件包名称](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#setPackage\(java.lang.String\))来控制谁能接收非粘性广播。此外，如果不需要向应用以外的组件发送广播，则可以使用 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData?hl=zh-cn)，它实现了[观察者模式](https://en.wikipedia.org/wiki/Observer_pattern)。

如需详细了解如何确保广播的安全，请参阅[广播概览](https://developer.android.com/guide/components/broadcasts?hl=zh-cn#security-and-best-practices)页面。

