---
title: Android-安全性-支持“直接启动”模式
date: 2026-1-23 14:30:50 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
description:
math: true
---
当设备已开机但用户尚未解锁设备时，Android 7.0 将在安全的“直接启动”模式下运行。为支持此模式，系统为数据提供了两个存储位置：

- **凭据加密存储空间**，这是默认存储位置，仅在用户解锁设备后可用。
- **设备加密存储空间**，该存储位置在“直接启动”模式下和用户解锁设备后均可使用。

默认情况下，应用不会在“直接启动”模式下运行。如果您的应用需要在“直接启动”模式下执行操作，您可以注册能在此模式下运行的应用组件。需要在“直接启动”模式下运行的一些常见应用用例包括：

- 已安排通知的应用，如闹钟应用。
- 提供重要用户通知的应用，如短信应用。
- 提供无障碍服务的应用，如 Talkback。

如果应用在“直接启动”模式下运行时需要访问数据，请使用设备加密存储空间。设备加密存储空间包含使用密钥加密的数据，该密钥只有在设备成功执行启动时验证后才可用。

对于必须使用与用户凭据（如 PIN 码或密码）关联的密钥加密的数据，请使用凭据加密存储空间。凭据加密存储空间在用户成功解锁设备后才可用，直到用户再次重启设备。如果用户在解锁设备后启用锁定屏幕，凭据加密存储空间依然可用。

## 请求在“直接启动”模式下运行

应用必须先向系统注册其组件，然后才能在“直接启动”模式下运行或访问设备加密存储空间。应用通过将组件标记为加密感知来向系统注册。如需将您的组件标记为加密感知，请在清单中将 `android:directBootAware` 属性设为 true。

当设备重启后，加密感知组件可以注册以接收来自系统的 [`ACTION_LOCKED_BOOT_COMPLETED`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#ACTION_LOCKED_BOOT_COMPLETED) 广播消息。此时，设备加密存储空间可用，您的组件可以执行需要在“直接启动”模式下运行的任务，例如触发已设定的闹铃。

以下代码段示例说明了如何在应用清单中将 [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=zh-cn) 注册为加密感知并为 `ACTION_LOCKED_BOOT_COMPLETED` 添加 intent 过滤器：

```xml
<receiver
  android:directBootAware="true" >
  ...
  <intent-filter>
    <action android:name="android.intent.action.LOCKED_BOOT_COMPLETED" />
  </intent-filter>
</receiver>
```

在用户解锁设备后，所有组件均可访问设备加密存储空间和凭据加密存储空间。

## 访问设备加密存储空间

如需访问设备加密存储空间，请通过调用 [`Context.createDeviceProtectedStorageContext()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createDeviceProtectedStorageContext\(\)) 创建另一个 [`Context`](https://developer.android.com/reference/android/content/Context?hl=zh-cn) 实例。通过此上下文发出的所有存储 API 调用均访问设备加密存储空间。以下示例会访问设备加密存储空间并打开现有的应用数据文件：

[Kotlin](https://developer.android.com/privacy-and-security/direct-boot?hl=zh-cn#kotlin)
```kotlin
val directBootContext: Context = appContext.createDeviceProtectedStorageContext()
// Access appDataFilename that lives in device encrypted storage
val inStream: InputStream = directBootContext.openFileInput(appDataFilename)
// Use inStream to read content...
```
[Java](https://developer.android.com/privacy-and-security/direct-boot?hl=zh-cn#java)

```java
Context directBootContext = appContext.createDeviceProtectedStorageContext();
// Access appDataFilename that lives in device encrypted storage
FileInputStream inStream = directBootContext.openFileInput(appDataFilename);
// Use inStream to read content...
```

请只将设备加密存储空间用于在“直接启动”模式下必须可以访问的信息。请勿将设备加密存储空间用作通用加密存储空间。对于私密用户信息，或在“直接启动”模式下不需要的加密数据，请使用凭据加密存储空间。

## 接收用户解锁通知

当用户在重启后解锁设备时，应用可以切换至访问凭据加密存储空间，并使用依赖用户凭据的常规系统服务。

为了在重启后用户解锁设备时收到通知，请从正在运行的组件注册 `BroadcastReceiver` 以监听解锁通知消息。在用户重启后解锁设备时：

- 如果应用具有需要立即获得通知的前台进程，请监听 [`ACTION_USER_UNLOCKED`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#ACTION_USER_UNLOCKED) 消息。
- 如果应用仅使用可以对延迟通知执行操作的后台进程，请监听 [`ACTION_BOOT_COMPLETED`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#ACTION_BOOT_COMPLETED) 消息。

您可以通过调用 [`UserManager.isUserUnlocked()`](https://developer.android.com/reference/android/os/UserManager?hl=zh-cn#isUserUnlocked\(\)) 直接查询用户是否已解锁设备。

## 迁移现有数据

如果用户将其设备更新为使用“直接启动”模式，您可能需要将现有数据迁移到设备加密存储空间。使用 [`Context.moveSharedPreferencesFrom()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#moveSharedPreferencesFrom\(android.content.Context,%20java.lang.String\)) 和 [`Context.moveDatabaseFrom()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#moveDatabaseFrom\(android.content.Context,%20java.lang.String\))（将目标上下文作为方法调用方，将来源上下文作为参数），可以在凭据加密存储空间与设备加密存储空间之间迁移偏好设置和数据库数据。

请勿将私密用户信息（如密码或授权令牌）从凭据加密存储空间迁移到设备加密存储空间。请自行判断要从凭据加密存储空间向设备加密存储空间迁移哪些数据。在某些情况下，您可能需要在这两种加密存储空间中管理单独的数据集。

## 测试加密感知应用

在启用“直接启动”模式的情况下测试您的加密感知应用。

如果设置了锁屏凭据（PIN 码、图案或密码），大多数搭载最新版 Android 的设备都会启用“直接启动”模式。具体而言，所有使用文件级加密的设备都会如此。如需检查设备是否使用了文件级加密，请运行以下 shell 命令：

```bash
adb shell getprop ro.crypto.type
```

如果输出为 `file`，则表示设备启用了文件级加密。

对于默认不使用文件级加密的设备，可能有其他用于测试“直接启动”模式的选项：

- 某些使用完整磁盘加密 (`ro.crypto.type=block`) 且搭载 Android 7.0 到 Android 12 的设备可以转换为文件级加密。您可以采用下列两种方法：
    
    >**警告：** 无论采用哪种方法转换为文件级加密，都会擦除设备上的所有用户数据。
    
    - 在设备上，如果您尚未启用**开发者选项**，请依次选择**设置 > 关于手机**，然后点按 **build 号**七次，将其启用。然后，依次前往**设置 > 开发者选项**，并选择**转换为文件级加密**。
    - 或者，运行以下 shell 命令：
        
```bash
	adb reboot-bootloader
	fastboot --wipe-and-use-fbe
```
        
- 搭载 Android 13 或更低版本的设备支持“直接启动”模拟模式，该模式使用文件权限来模拟加密文件被锁定和解锁时的加密效果。仅在开发期间使用模拟模式，因为可能会导致数据丢失。如需启用“直接启动”模拟模式，请在设备上设置锁定模式，如果在设置锁定模式时系统提示使用安全启动屏幕，请选择“不用了”，然后运行以下 shell 命令：
    
```bssh
    adb shell sm set-emulate-fbe true
```
    
如需关闭“直接启动”模拟模式，请运行以下 shell 命令：
    
```bash
    adb shell sm set-emulate-fbe false
```
    
运行其中任一命令都会导致设备重启。
    

## 检查设备政策加密状态

设备管理应用可以使用 [`DevicePolicyManager.getStorageEncryptionStatus()`](https://developer.android.com/reference/android/app/admin/DevicePolicyManager?hl=zh-cn#getStorageEncryptionStatus\(\)) 检查设备目前的加密状态。

如果您的应用以低于 Android 7.0 (API 24) 的 API 级别为目标平台，则当设备使用完整磁盘加密或带“直接启动”的文件级加密时，`getStorageEncryptionStatus()` 会返回 [`ENCRYPTION_STATUS_ACTIVE`](https://developer.android.com/reference/android/app/admin/DevicePolicyManager?hl=zh-cn#ENCRYPTION_STATUS_ACTIVE)。在这两种情况下，数据在静态存储时始终以加密形式保存。

如果您的应用以 Android 7.0 (API 24) 或更高版本为目标平台，则当设备使用完整磁盘加密时，`getStorageEncryptionStatus()` 会返回 `ENCRYPTION_STATUS_ACTIVE`。如果设备使用带“直接启动”的文件级加密，则其返回 [`ENCRYPTION_STATUS_ACTIVE_PER_USER`](https://developer.android.com/reference/android/app/admin/DevicePolicyManager?hl=zh-cn#ENCRYPTION_STATUS_ACTIVE_PER_USER)。

如果面向 Android 7.0 构建设备管理应用，请务必同时检查 `ENCRYPTION_STATUS_ACTIVE` 和 `ENCRYPTION_STATUS_ACTIVE_PER_USER` 以确定设备是否已加密。

## 更多代码示例

[DirectBoot](https://github.com/android/security-samples/tree/main/DirectBoot/) 示例进一步演示了如何使用本页介绍的 API。