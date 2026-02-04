---
title: Android-隐私权-隐私权准则
date: 2026-1-29 10:57:43 +0800
categories:
  - Android
  - Privacy
tags:
  - Android
  - Privacy
description: 了解常见的隐私权准则和最佳实践。
math: true
---
# 隐私权准则

Android 致力于帮助用户充分利用最新的创新技术，同时将用户的安全和隐私视为第一要务。您可以使用此页面上的核对清单，了解常见的隐私权准则和最佳实践。

本页面中介绍的一些最佳实践也会显示在[备忘单](https://developer.android.com/privacy-and-security/about?hl=zh-cn#privacy-cheatsheet)中。

## 核对清单：尽量减少权限请求

确保公开透明并让用户自主控制应用的使用体验，从而赢得用户信任。

- **请求授予应用功能所需的最基本权限：** 在对应用进行重大更改时，请查看[所请求的权限](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn#avoid_requesting_unnecessary_permissions)，以确认应用功能是否仍然需要这些权限。
    - 较高版本的 Android 往往会引入注重隐私保护的方法来访问数据，这些方法不需要权限。如需了解详情，请参阅[评估应用是否需要声明权限](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)。
    - 如果您的应用是在 Google Play 上分发的，则可使用 [Android Vitals](https://developer.android.com/topic/performance/vitals/permissions?hl=zh-cn#use_android_vitals_to_gauge_user_perceptions) 来获取拒绝授予应用所请求权限的用户所占的百分比。请使用此数据找出所需权限最常被拒绝的功能，并重新评估其设计。
- **说明应用中的功能为何需要某项权限：** 按照[推荐流程](https://developer.android.com/training/permissions/requesting?hl=zh-cn#explain)执行此操作。仅在需要时（而不是在应用启动时）才请求权限，以便用户清楚地了解您的应用所需要的权限。
- **请注意，用户或系统可能会多次拒绝该权限：** Android 会尊重用户的选择，忽略来自同一应用的权限请求。
- **未经授权情况下进行适当降级：** 当用户拒绝或撤消某项权限时，应用应进行适当降级，例如，如果用户未授予麦克风权限，则禁用语音输入。
- **撤消不必要的权限：** 当更新应用时，请[撤消应用不再需要的任何运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn#remove-access)。
- **了解 SDK 或库所需的权限：** 如果您使用的 SDK 或库会访问受危险权限保护的数据，用户通常会认为是您的应用需要相应的访问权限。请务必[了解您的 SDK 所需的权限及其原因](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn#sdk-libraries)。

## 核对清单：尽量减少对位置信息的使用

有关用户所在位置的数据属于敏感内容；请尽可能避免使用位置数据。如果必须使用位置信息服务，请采取措施尽量减少对位置数据的收集。请使用以下核对清单来尽量减少应用对位置信息的使用。

- **在没有位置数据的情况下进行适当降级：** 在 Android 10（API 级别 29）及更高版本中，用户可以限制应用仅在使用时才能访问位置信息。请将您的应用设计为在用户未授予“不间断允许”访问位置信息的权限时，以适当的方式进行功能降级。
- **使用附近的蓝牙或 Wi-Fi 设备：** 如果您的应用需要通过蓝牙或 Wi-Fi 将用户的设备与附近的设备配对，请使用不需要位置信息权限的[配套设备管理器](https://developer.android.com/guide/topics/connectivity/companion-device-pairing?hl=zh-cn)。详细了解[蓝牙](https://developer.android.com/guide/topics/connectivity/bluetooth/permissions?hl=zh-cn)和 [Wi-Fi](https://developer.android.com/guide/topics/connectivity/wifi-permissions?hl=zh-cn) 权限。
- **尽可能使用粗略位置信息精确度：** 检查您的应用所需的[位置信息精细度级别](https://developer.android.com/training/location/permissions?hl=zh-cn#accuracy)。粗略位置信息访问权限足以满足大多数与位置相关的用途。
- **仅在必要时在后台访问位置信息：** 如果您的应用需要在后台访问位置信息（例如使用地理围栏时），请采用对用户而言显而易见的方式来实现。详细了解[使用后台位置信息的注意事项](https://developer.android.com/training/location/background?hl=zh-cn)。
- **在用户进入应用界面时访问位置数据：** 这有助于用户更好地了解应用为何请求获取位置信息。
- **请勿在后台启动前台服务：** 应考虑从通知中启动应用，然后在应用界面变为可见时执行需要访问位置信息的代码。如果在用户离开应用界面后，应用必须继续访问位置信息才能执行用户启动的持续性任务，请在应用进入后台之前[启动前台服务](https://developer.android.com/guide/components/services?hl=zh-cn#Types-of-services)。

## 核对清单：安全处理数据

>**注意：** 您可以在 Google Play 开发者政策中心的[用户数据文章](https://play.google.com/about/privacy-security-deception/user-data/?hl=zh-cn#!?zippy_activeEl=personal-sensitive#personal-sensitive)页面详细了解什么是敏感数据。

以透明、安全、周到的方式处理敏感数据。为了在应用中更为安全地处理用户数据，请参考以下核对清单。

- **审核数据访问：** 在 Android 11（API 级别 30）及更高版本中，通过[执行数据访问审核](https://developer.android.com/security-and-privacy/data/audit-access?hl=zh-cn)，可以获取应用及其依赖项对用户私密数据的访问情形方面的分析数据，从而轻松识别意外数据访问。
    
- **声明软件包可见性需求：** 如果您的应用以 Android 11 或更高版本为目标平台，在默认情况下，系统会让部分应用对您的应用不可见。了解如何[让上述其他应用对您的应用可见](https://developer.android.com/training/package-visibility/declaring?hl=zh-cn)。
    
- **支持分区存储：** 为了让用户更好地管理自己的文件并减少混乱，以 Android 10（API 级别 29）及更高版本为目标平台的应用会自动获得对外部存储空间的分区访问权限（[即分区存储](https://developer.android.com/training/data-storage?hl=zh-cn#scoped-storage)）。 此类应用只能访问自己创建的目录和媒体文件。了解如何[迁移到分区存储](https://developer.android.com/training/data-storage/use-cases?hl=zh-cn)。
    
- **使用用户可重置的标识符：** 为了保护用户的隐私，请使用符合您应用场景且限制性最强的标识符（请参阅[本文档中可重置的标识符核对清单](https://developer.android.com/privacy-and-security/about?hl=zh-cn#resettable-identifiers)。
    
- **提供醒目披露声明和征求用户同意：** 请遵循 [Google Play 用户数据政策最佳实践](https://support.google.com/googleplay/android-developer/answer/11150561?hl=zh-cn)，向用户提供醒目披露声明和征求用户同意。
    
- **声明应用的数据使用方式：**[正确填写 Google Play 管理中心数据安全表单](https://developer.android.com/security-and-privacy/data/declare-data-use?hl=zh-cn)，其中向用户说明您的应用会收集和共享哪些类型的用户数据。
    
- **安全地将敏感数据传递给其他应用：** 使用显式 intent 将敏感数据传递给其他应用。[通过授予一次性数据访问权限](https://developer.android.com/topic/security/best-practices?hl=zh-cn#permissions-share-data)来进一步限制其他应用的访问权限。
    
- **请勿在 Logcat 消息或日志文件中包含敏感数据：**[了解详情](https://developer.android.com/training/articles/security-tips?hl=zh-cn#UserData)。
    
>**注意：** Jetpack 提供了多个库来提升应用数据的安全性。如需详细了解如何使用 [Jetpack Security 库](https://developer.android.com/topic/security/data?hl=zh-cn)和 [Jetpack Preferences 库](https://developer.android.com/guide/topics/ui/settings/use-saved-values?hl=zh-cn)，请参阅相关指南。

## 核对清单：使用可重置的标识符

尊重用户的隐私权并使用可重置的标识符。如需了解详情，请参阅[唯一标识符最佳实践](https://developer.android.com/training/articles/user-data-ids?hl=zh-cn)。

- **请勿访问 IMEI 或设备序列号：** 这些标识符是永久性的。对于以 Android 10（API 级别 29）或更高版本为目标平台的应用，如果尝试访问这些标识符，会导致 [`SecurityException`](https://developer.android.com/reference/java/lang/SecurityException?hl=zh-cn)。
    
- **只针对用户分析或广告用例使用广告 ID：** 始终尊重用户针对[广告跟踪](https://support.google.com/googleplay/android-developer/answer/6048248?hl=zh-cn)的个性化偏好设置。**重要提示：** 这是 Google Play 要求必须做到的。
    
- **使用私密存储的 GUID：** 对于绝大多数非广告用例，请[使用作用域仅限于应用的私密存储全局唯一 ID (GUID)](https://developer.android.com/training/articles/user-data-ids?hl=zh-cn#signed-out-anon-user-analytics)。
    
- **针对您拥有的应用使用 SSAID：** 在您拥有的应用之间共享状态，而无需要求用户登录账号，请使用安全设置 Android ID (SSAID)。详细了解如何[在不同应用之间保存未登录账号的用户的偏好设置](https://developer.android.com/training/articles/user-data-ids?hl=zh-cn#signed-out-user-prefs)。
    

## 核对清单：支持面向用户的隐私保护功能

以透明、安全、周到的方式处理敏感数据。为了确保应用更为安全地处理用户数据，请参考以下核对清单。

- **提供访问敏感信息的理由：** 在 Android 12（API 级别 31）及更高版本中，用户可以访问系统设置中的隐私信息中心，以详细了解应用在何时访问了位置、麦克风和摄像头信息。详细了解如何[向用户提供此说明](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#privacy-dashboard)。
    
- **提示用户停用应用休眠功能：** 如果用户数月未与以 Android 11（API 级别 30）或更高版本为目标平台的应用进行过互动，系统会将该应用置于休眠状态。了解[应用休眠以及如何要求用户停用此功能](https://developer.android.com/topic/performance/app-hibernation?hl=zh-cn)。
    
- **安全地将敏感数据传递给其他应用：** 如需将敏感数据传递给其他应用，请使用[显式 intent](https://developer.android.com/guide/components/intents-filters?hl=zh-cn#Types)。[通过授予一次性数据访问权限](https://developer.android.com/topic/security/best-practices?hl=zh-cn#permissions-share-data)来进一步限制其他应用的访问权限。
    
- **直观表明您的应用正在采集音频或图像：** 即使应用在前台运行，也应显示一个实时指示器，表明您正在通过麦克风或摄像头进行采集。当应用在后台运行时，Android 9（API 级别 28）及更高版本将[不允许应用访问麦克风或相机](https://developer.android.com/about/versions/pie/android-9.0-changes-all?hl=zh-cn#privacy-changes-all)。
    

## 隐私保护备忘单

隐私保护备忘单是对 Android 中一些最实用的隐私 API 的快速参考，并列出了在设计应用时最先想到的最佳实践。

您也可以下载 PDF 格式的备忘单：

- [浅色模式 PDF](https://developer.android.com/static/privacy-and-security/images/cheat-sheet-light.pdf?hl=zh-cn)
- [深色模式 PDF](https://developer.android.com/static/privacy-and-security/images/cheat-sheet-dark.pdf?hl=zh-cn)

![Privacy cheat sheet](https://rd-wang.github.io/assets/img/android/privacy/cheat-sheet.svg)
