---
title: Android-隐私权-尽量减少权限请求
date: 2026-2-4 11:18:51 +0800
categories:
  - Android
  - Privacy
  - Permission
tags:
  - Android
  - Privacy
  - Permission
description: 建议您尽量少在应用内使用权限。这有助于用户发现和使用能提供安全可靠的用户环境的优质应用。
math: true
---
为了[提高应用质量](https://android-developers.googleblog.com/2022/10/raising-bar-on-technical-quality-on-google-play.html)和保护用户隐私，我们建议您尽量少在应用内使用权限。这有助于用户发现和使用能提供安全可靠的用户环境的优质应用。

向用户请求权限会中断用户体验流程，而且用户可能会拒绝您的请求。此外，每次声明新权限时，您都必须[检查您的应用是如何请求和分享用户数据](https://developer.android.com/guide/topics/data/collect-share?hl=zh-cn)的。某些[特别敏感的权限和 API](https://support.google.com/googleplay/android-developer/answer/9888170?hl=zh-cn) 会要求您提供应用内披露声明，以说明您对数据的访问、收集、使用和分享方式。

您可以通过多种替代方案来尽可能减少使用权限：

- 如果您的应用只需要大致位置信息，请声明提供粗略位置信息（而非确切位置信息）的权限。
- 调用能让您的应用在不声明权限的情况下执行所需功能的 API。
- 调用特定的 intent 或事件处理脚本来执行功能，而不是声明权限。
- 系统会为不同的文件操作提供[内置协定](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts?hl=zh-cn)，并且也支持[自定义协定](https://developer.android.com/training/basics/intents/result?hl=zh-cn#custom)。

如果您必须声明某项权限，请始终[尊重用户的决定](https://developer.android.com/training/permissions/requesting?hl=zh-cn#handle-denial)，并提供一种方式让应用体验优雅降级。

本页将介绍您的应用可在不声明任何权限的情况下实现的几个用例。

## 显示附近的地点

您的应用可能需要了解用户的大致位置。有了此信息，应用就能显示位置感知信息，比如附近的餐馆。

在某些用例中，您的应用只需要粗略估算的设备位置信息。在这类情况下，请根据应用需要位置感知信息的频率，执行以下某项操作：

- 如果您的应用频繁需要位置信息，请声明 [`ACCESS_COARSE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_COARSE_LOCATION) 权限。该权限提供位置信息服务估算的设备位置信息，如介绍[大致位置信息精确度](https://developer.android.com/training/location/permissions?hl=zh-cn#accuracy)的文档中所述。
- 如果您的应用不常需要获取位置信息，或只需要获取一次，请考虑改为让用户输入地址或邮政编码。

在其他用例中，您的应用需要更精确地估算的设备位置信息。只有在这些情况下，才可以声明 [`ACCESS_FINE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_FINE_LOCATION) 权限。

## 创建和访问文件

Android 允许您不必声明任何与存储或传感器相关的权限就能创建和访问文件。

### 打开媒体文件

您的应用可能会允许用户选择其照片和视频以实现某些目的，例如用作消息附件或个人资料照片。

如需支持此功能，请使用[照片选择器](https://developer.android.com/training/data-storage/shared/photopicker?hl=zh-cn)。照片选择器不需要任何运行时权限。当用户与照片选择器互动以选择要通过您的应用分享的照片或视频时，系统会授予对所选媒体文件相关联 URI 的临时读取权限。

如果您的应用需要在不使用照片选择器的情况下访问媒体文件，那么您无需声明任何存储权限：

- 如果您要访问您的应用创建的媒体文件，此时该应用已经具有访问[媒体库](https://developer.android.com/training/data-storage/shared/media?hl=zh-cn#media_store)中的这些文件的权限。
- 如果您要访问其他应用创建的媒体文件，请[使用存储访问框架](https://developer.android.com/training/data-storage/shared/documents-files?hl=zh-cn)。

### 打开文档

您的应用可能会显示用户在您的应用或其他应用中创建的文档。一个常见的示例是文本文件。

在这种情况下，请仅声明 [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE) 以便与旧设备兼容。将 `android:maxSdkVersion` 设为 `28`。

根据创建文档的是哪个应用，执行下列操作之一：

- 如果用户是在您的应用中创建的文档，请[直接访问该文档](https://developer.android.com/training/data-storage/app-specific?hl=zh-cn#external-access-files)。
- 如果用户是在其他应用中创建的文档，请使用[存储访问框架](https://developer.android.com/training/data-storage/shared/documents-files?hl=zh-cn)。

### 拍摄照片

用户可能会在您的应用中使用预安装的系统相机应用来拍摄照片。

在这种情况下，请勿声明 `CAMERA` 权限，而是应调用 [`ACTION_IMAGE_CAPTURE`](https://developer.android.com/reference/android/provider/MediaStore?hl=zh-cn#ACTION_IMAGE_CAPTURE) intent 操作。

> **注意：** 如果您的应用声明了 Manifest.permission.CAMERA 权限，但未被授予，则该操作会导致 `SecurityException`。

### 录制视频

用户可能会在您的应用中使用预安装的系统相机应用来录制视频。

在这种情况下，请勿声明 `CAMERA` 权限，而是应调用 [`ACTION_VIDEO_CAPTURE`](https://developer.android.com/reference/android/provider/MediaStore?hl=zh-cn#ACTION_VIDEO_CAPTURE) intent 操作。

> **注意：** 如果您的应用声明了 Manifest.permission.CAMERA 权限，但未被授予，则该操作会导致 `SecurityException`。

## 识别正在运行应用的某个实例的设备

您的应用的特定实例可能需要知道它正在哪个设备上运行。对于采用设备专属偏好设置或提供消息功能（如面向电视设备和穿戴式设备提供不同的播放列表）的应用而言，此信息非常有用。

在这种情况下，请勿直接获取设备的 IMEI。实际上，从 Android 10 开始，您已无法这样做。请改用以下方法之一：

- 使用[实例 ID](https://developers.google.com/instance-id/guides/android-implementation?hl=zh-cn) 库获取应用实例的唯一设备标识符。
- 创建您自己的标识符，将其存储在您的应用存储空间。使用基本系统函数，例如 [`randomUUID()`](https://developer.android.com/reference/java/util/UUID?hl=zh-cn#randomUUID\(\))。

## 通过蓝牙与设备配对

通过利用蓝牙将数据传输到其他设备，您的应用或许能提供更好的体验。

如需支持此功能，请勿声明 `ACCESS_FINE_LOCATION`、`ACCESS_COARSE_LOCATIION` 或 `BLUETOOTH_ADMIN` 权限，而是改用[配套设备配对](https://developer.android.com/guide/topics/connectivity/companion-device-pairing?hl=zh-cn)。

## 自动输入支付卡号码

Google Play 服务提供了一个库，让您能够实现自动输入支付卡号码。您可以使用[借记卡和信用卡识别](https://developers.google.com/pay/payment-card-recognition/debit-credit-card-recognition?hl=zh-cn)库，而不是声明 `CAMERA` 权限。

## 管理通话和短信

Android 和 Google Play 服务提供的库让您不必声明任何与通话或短信相关的权限就能管理通话和短信。

### 自动输入一次性密码

为了简化双重身份验证工作流，您的应用可能会自动输入发送给用户设备的动态密码以验证用户身份。

如需在由 Google Play 服务提供支持的设备上支持此功能，请勿声明 `READ_SMS` 权限，而是改用 [SMS Retriever API](https://developers.google.com/identity/sms-retriever/overview?hl=zh-cn)。

在其他设备上，如果您的应用以 Android 8.0（API 级别 26）或更高版本为目标平台，请使用 [`createAppSpecificSmsToken()`](https://developer.android.com/reference/android/telephony/SmsManager?hl=zh-cn#createAppSpecificSmsToken\(android.app.PendingIntent\)) 生成应用专用令牌。然后，将此令牌传递给可发送验证短信的其他应用或服务。

### 自动输入用户的电话号码

为了提供更高效的销售或支持，您的应用可能会允许自动输入用户设备的电话号码。

如需在由 Google Play 服务提供支持的设备上支持此功能，请勿声明 `READ_PHONE_STATE` 权限，而是改用[电话号码提示](https://developers.google.com/identity/phone-number-hint/android?hl=zh-cn)库。

### 过滤来电

为了给用户最大限度地减少不必要的干扰，您的应用可能会过滤掉垃圾来电。

如需支持此功能，请勿声明 `READ_PHONE_STATE` 权限，而是改用 [`CallScreeningService`](https://developer.android.com/reference/android/telecom/CallScreeningService?hl=zh-cn) API。

### 拨打电话

您的应用可能会提供通过点按某个联系人的信息拨打电话的功能。

如需支持此功能，请使用 [`ACTION_DIAL`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#ACTION_DIAL) intent 操作，而不是 `ACTION_CALL` 操作。`ACTION_CALL` 需要安装时权限 `CALL_PHONE`，这样可以阻止无法拨打电话的设备（比如某些平板电脑）安装您的应用。

## 在应用中断运行时暂停媒体

在用户接听电话或用户配置的闹钟触发时，您的应用应暂停播放所有媒体，直到其重新获得音频焦点再恢复播放。

如需支持此功能，请勿声明 `READ_PHONE_STATE` 权限，而是改为实现 [`onAudioFocusChange()`](https://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener?hl=zh-cn#onAudioFocusChange\(int\)) 事件处理脚本，它会在系统转换其音频焦点时自动运行。 详细了解如何[实现音频焦点](https://developer.android.com/guide/topics/media-apps/audio-focus?hl=zh-cn)。

## 扫描条形码

Android 支持 [Google Code Scanner API](https://developers.google.com/ml-kit/vision/barcode-scanning/code-scanner?hl=zh-cn)，此 API 由 Google Play 服务提供支持，可让您在不声明任何相机权限的情况下解码条形码。此 API 有助于保护用户隐私。使用此 API，您就很少需要为条形码扫描用例创建自定义界面。

此 API 会扫描条形码，并且仅会将扫描结果返回给您的应用。图片将在设备端进行处理，Google 不会存储任何数据或扫描结果。

如果您的应用需要支持复杂的用例或条形码格式，或者需要自定义界面，请改用[机器学习套件条形码扫描 API](https://developers.google.com/ml-kit/vision/barcode-scanning/android?hl=zh-cn)。

## 重置未使用的权限

Android 提供了多种方法来将未使用的运行时权限重置为默认的拒绝状态。

阅读[设计指南](https://developer.android.com/training/permissions/requesting?hl=zh-cn#reset-unused-permissions)。

## 请求运行时权限

如果您已评估确定应用需要声明和请求运行时权限，请按照特定工作流执行此操作。

阅读[设计指南](https://developer.android.com/training/permissions/requesting?hl=zh-cn#workflow_for_requesting_permissions)。

## 说明您的应用为何需要获取权限

使用 `requestPermissions()` 会显示一个对话框，其中显示应用想要使用哪些权限，但没有说明原因，这可能会让用户感到困惑。

如需详细了解如何以及何时显示此对话框，并获取相关建议，请参阅[设计指南](https://developer.android.com/training/permissions/requesting?hl=zh-cn#explain)。

## 处理权限被拒情况

在用户选择拒绝某个权限之前和之后，您的应用应能帮助用户了解拒绝该权限可能带来的影响。

阅读[设计指南](https://developer.android.com/training/permissions/requesting?hl=zh-cn#handle-denial)。