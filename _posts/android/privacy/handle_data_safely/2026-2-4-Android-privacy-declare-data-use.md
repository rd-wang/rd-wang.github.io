---
title: Android-隐私权-声明应用的数据使用方式
date: 2026-2-4 11:35:00 +0800
categories:
  - Android
  - Privacy
tags:
  - Android
  - Privacy
description: Play 管理中心在应用内容页面上提供了数据安全表单。在此表单中，您将向用户说明您的应用会收集和分享哪些类型的用户数据。这些信息有助于提高数据的透明度和用户的信任度。
math: true
---
> **注意**：本文补充介绍了如何在向 Google Play 发布应用时[填写 Google Play 的“数据安全”部分要求提供的信](https://support.google.com/googleplay/android-developer/answer/10787469?hl=zh-cn)。建议您先参阅这篇帮助中心文章，然后再查看本文。

Play 管理中心在**应用内容**页面上提供了**数据安全**表单。在此表单中，您将向用户说明您的应用会收集和分享哪些类型的用户数据。这些信息有助于提高数据的透明度和用户的信任度。

为帮助您填写此表单，本指南将举例说明应用代码中哪些位置的代码可能用于为应用收集不同类型的用户数据。具体而言，本指南将举例说明应用可能用于收集或分享特定类型用户数据的权限声明和 API 调用，这些特定类型的数据要求您在**数据安全**表单中声明应用需要收集或分享数据。

本指南采用以下格式：

- 主标题列出了**数据安全**表单中可用的不同类别的数据类别，以及您的应用或应用所含库可能会访问特定类别相关用户数据的一部分方式。
- 子标题列出了此表单中提供的不同数据类型，提醒您某些数据类型属于特定类别。

尽管我们提供此指南是为了帮助您更轻松地填写**数据安全**表单，但您需要自行负责在应用的 Play 商品详情中给出完整且准确的声明。只有您拥有填写**数据安全**表单所需的全部信息。开发者如何根据其对数据的用法和相关做法收集或处理用户数据由他们自己决定，Google 不能代表开发者做出相关决定。开发者应妥善处理其应用收集或分享的任何用户数据。当我们发现您的应用行为与您的声明之间存在差异时，可能会采取适当的措施，包括违规处置措施。

本指南中的详细信息可能会随着我们继续与开发者合作并不断改善开发者和用户体验而发生变化。如需了解详情，请参阅此[博文](https://android-developers.googleblog.com/2021/10/launching-data-safety-in-play-console.html)。

**注意**：您的应用包含的第三方 SDK 和库可能会访问用户数据。如果您应用中的第三方 SDK 或库会收集或分享用户数据，您必须在**数据安全**表单中反映这种收集和分享行为。

## 通用指南

除了下文几部分中列出的具体建议之外，可能还有一些更为宽泛的指标可以表明您的应用或应用所含库会收集或共享用户数据。这些指标包括但不限于：

- 用于隐藏文本输入的界面组件，例如密码输入字段。
- 用于请求用户[使用生物识别技术进行身份验证](https://developer.android.com/training/sign-in/biometric-auth?hl=zh-cn)的工作流。
- 用于提示用户输入用户数据、确认提交内容或作出选择的表单和提醒。
- 用于控制应用中 [`WebView`](https://developer.android.com/guide/webapps/webview?hl=zh-cn) 元素行为的代码。
- 对[自动填充框架](https://developer.android.com/guide/topics/text/autofill-optimize?hl=zh-cn#hints)的支持。
- [数据访问审核](https://developer.android.com/guide/topics/data/audit-access?hl=zh-cn) API 的使用。

## 位置信息

您的应用或应用所含库可以通过多种方式访问与位置信息相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`ACCESS_COARSE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_COARSE_LOCATION)
    - [`ACCESS_FINE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_FINE_LOCATION)
    - [`ACCESS_MEDIA_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_MEDIA_LOCATION)
- 通过 IP 地址或接入点名称推导位置信息。

**注意：**在搭载 Android 14 及更高版本的设备上，有关位置数据收集和共享做法的详细信息显示在[运行时系统权限对话框](https://developer.android.com/training/permissions/requesting?hl=zh-cn)中。详细了解如何在 Android 14 中[以更显眼的方式显示数据安全信息](https://developer.android.com/about/versions/14/changes/data-safety?hl=zh-cn)。

### 大致位置

用户或设备的实际位置，范围大于或等于 3 平方公里，例如用户所在的城市或 Android 的 `ACCESS_COARSE_LOCATION` 权限提供的位置。

### 确切位置信息

用户或设备的实际位置，范围小于 3 平方公里，例如 Android 的 `ACCESS_FINE_LOCATION` 权限提供的位置。

## 个人信息

您的应用或应用所含库可以通过多种方式访问与个人信息相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`BIND_AUTOFILL_SERVICE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BIND_AUTOFILL_SERVICE)
    - [`GET_ACCOUNTS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#GET_ACCOUNTS)
- 使用 [`AccountManager`](https://developer.android.com/reference/android/accounts/AccountManager?hl=zh-cn) API。

### 姓名

用户称呼自己的方式，例如用户的名字、姓氏或昵称。

### 电子邮件地址

用户的电子邮件地址。

### 用户 ID

与可识别身份的用户相关的用户 ID，例如账号 ID、账号或账号名称。

### 地址

用户的地址，例如邮寄地址或家庭住址。

### 电话号码

用户的电话号码。

除了[此部分的开头](https://developer.android.com/privacy-and-security/declare-data-use?hl=zh-cn#personal-info)列出的更宽泛的建议之外，可能还有一些更为具体的指标可以表明您的应用或应用所含库收集或共享用户的电话号码。 这些指标包括但不限于：

- 至少声明以下其中一项权限：
    - [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG)
    - [`READ_PHONE_NUMBERS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_PHONE_NUMBERS)
    - [`READ_PHONE_STATE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_PHONE_STATE)（如果应用以 Android 10（API 级别 29）或更低版本为目标平台）
    - [`READ_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_SMS)
- [请求用户同意](https://developer.android.com/guide/topics/permissions/default-handlers?hl=zh-cn#request-user-consent)让应用成为设备的默认拨号器应用或默认短信应用。
- [具有运营商权限](https://developer.android.com/reference/android/telephony/TelephonyManager?hl=zh-cn#hasCarrierPrivileges\(\))。

**注意**：除非您的应用满足一组特定要求，否则 Google Play 会限制应用使用某些通话记录权限。详细了解[短信或通话记录权限组的使用](https://support.google.com/googleplay/android-developer/answer/10208820?hl=zh-cn)政策。

### 种族和民族

有关用户的种族或民族的信息。

### 政治信仰或宗教信仰

有关用户的政治或宗教信仰的信息。

### 性取向

有关用户性取向的信息。

### 其他信息

任何其他个人信息。例如，用户的出生日期、性别认同或退伍军人身份。

## 财务信息

您的应用或应用所含库可以通过多种方式访问与财务信息相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 使用 Google Play 的[结算库](https://developer.android.com/google/play/billing?hl=zh-cn)。
- 使用 [Google Pay](https://developers.google.com/pay/api/android/overview?hl=zh-cn) API。
- 声明 [`BIND_AUTOFILL_SERVICE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BIND_AUTOFILL_SERVICE) 权限并使用[自动填充框架](https://developer.android.com/guide/topics/text/autofill-optimize?hl=zh-cn#hints)。

### 用户付款信息

与用户的金融账号相关的信息，如信用卡号。

### 交易记录

与用户进行的购买或交易相关的信息。

### 信用评分

与用户信用相关的信息，例如用户的信用记录或信用评分。

### 其他财务信息

任何其他财务信息。例如，用户的薪资或债务。

## 健康与健身

您的应用或应用所含库可以通过多种方式访问与健康和健身信息相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`ACTIVITY_RECOGNITION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACTIVITY_RECOGNITION)
    - [`BODY_SENSORS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BODY_SENSORS)
    - [`BODY_SENSORS_BACKGROUND`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BODY_SENSORS_BACKGROUND)
- 至少使用以下任一 API：
    - [Google Fit](https://developers.google.com/fit/android?hl=zh-cn)
    - [Sleep](https://developers.google.com/location-context/sleep?hl=zh-cn)

### 健康信息

与用户健康状况相关的信息，例如医疗记录或症状信息。

### 健身信息

与用户的健身情况相关的信息，例如锻炼或其他身体活动。

## 信息

您的应用或应用所含库可以通过多种方式访问与信息相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`READ_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_SMS)
    - [`RECEIVE_MMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_MMS)
    - [`RECEIVE_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_SMS)
    - [`RECEIVE_WAP_PUSH`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_WAP_PUSH)
    - [`SEND_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SEND_SMS)
    - `WRITE_SMS` 取决于开发者的使用情况

**注意**：除非您的应用满足一组特定要求，否则 Google Play 会限制应用使用某些短信权限。详细了解[短信或通话记录权限组的使用](https://support.google.com/googleplay/android-developer/answer/10208820?hl=zh-cn)政策。

### 电子邮件

用户的电子邮件，包括电子邮件主题行、发件人、收件人和电子邮件内容。

### 短信或彩信

用户的短信，包括发信人、收信人和短信内容。

### 其他信息

任何其他类型的信息。例如，即时消息或聊天内容。

## 照片或视频

您的应用或应用所含库可以通过多种方式访问与照片或视频相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 使用系统的[照片选择器](https://developer.android.com/training/data-storage/shared/photopicker?hl=zh-cn)。
- 至少声明以下其中一项权限：
    - [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE)
    - [`READ_MEDIA_IMAGES`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_MEDIA_IMAGES)
    - [`READ_MEDIA_VIDEO`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_MEDIA_VIDEO)
    - [`READ_MEDIA_AUDIO`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_MEDIA_AUDIO)
    - [`WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_EXTERNAL_STORAGE)
- 提供[媒体文件处理](https://developer.android.com/training/data-storage/use-cases?hl=zh-cn#handle-media-files)工作流。
- 录制 [HDR 视频内容](https://developer.android.com/training/camera2/hdr-video-capture?hl=zh-cn)。

### 照片

用户的照片。

### 视频

用户的视频。

## 音频文件

您的应用或应用所含库可以通过多种方式访问与音频文件相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`CAPTURE_AUDIO_OUTPUT`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#CAPTURE_AUDIO_OUTPUT)
    - [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECORD_AUDIO)
    - [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE)
    - [`WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_EXTERNAL_STORAGE)
- 提供[媒体文件处理](https://developer.android.com/training/data-storage/use-cases?hl=zh-cn#handle-media-files)工作流。

### 语音或录音

用户的语音，例如语音信息或录音。

### 音乐文件

用户的音乐文件。

### 其他音频文件

用户创建或用户提供的其他任何音频文件。

## 文件和文档

您的应用或应用所含库可以通过多种方式访问与文件和文档相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE)
    - [`WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_EXTERNAL_STORAGE)
    - [`MANAGE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#MANAGE_EXTERNAL_STORAGE)
- 提供[数据和文件存储](https://developer.android.com/training/data-storage?hl=zh-cn)工作流。
- 提供[数据备份](https://developer.android.com/guide/topics/data/backup?hl=zh-cn)工作流。

**注意**：除非您的应用满足一组特定要求，否则 Google Play 会限制应用使用 [`MANAGE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#MANAGE_EXTERNAL_STORAGE) 权限。详细了解[“所有文件访问权限”(MANAGE_EXTERNAL_STORAGE) 的使用](https://support.google.com/googleplay/android-developer/answer/10467955?hl=zh-cn)相关政策。

## 日历

您的应用或应用所含库可以通过多种方式访问与用户日历相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- [`READ_CALENDAR`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALENDAR)
- [`WRITE_CALENDAR`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_CALENDAR)

## 通讯录

您的应用或应用所含库可以通过多种方式访问与用户通讯录相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`ACCEPT_HANDOVER`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCEPT_HANDOVER)
    - [`ADD_VOICEMAIL`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ADD_VOICEMAIL)
    - [`ANSWER_PHONE_CALLS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ANSWER_PHONE_CALLS)
    - [`CALL_PHONE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#CALL_PHONE)
    - [`PROCESS_OUTGOING_CALLS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#PROCESS_OUTGOING_CALLS)
    - [`READ_CALL_LOG`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CALL_LOG)
    - [`READ_CONTACTS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CONTACTS)
    - [`READ_PHONE_NUMBERS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_PHONE_NUMBERS)
    - [`READ_PHONE_STATE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_PHONE_STATE)
    - [`READ_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_SMS)
    - [`RECEIVE_MMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_MMS)
    - [`RECEIVE_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_SMS)
    - [`RECEIVE_WAP_PUSH`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECEIVE_WAP_PUSH)
    - [`SEND_SMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SEND_SMS)
    - [`WRITE_CONTACTS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_CONTACTS)

## 应用活动

您的应用或应用所含库可以通过多种方式访问与用户应用活动相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 绑定到系统服务，例如 [`AccessibilityService`](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService?hl=zh-cn) 或 [`TextService`](https://developer.android.com/reference/android/service/textservice/package-summary?hl=zh-cn)。
- 声明 [`QUERY_ALL_PACKAGES`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#QUERY_ALL_PACKAGES) 权限并调用 [`getInstalledApplications()`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#getInstalledApplications\(int\)) 或类似方法。
- 创建搜索界面。
- 支持 [Google 快捷方式集成库](https://developer.android.com/guide/topics/ui/shortcuts/creating-shortcuts?hl=zh-cn#gsi-library)。
- 使用 [`Instrumentation`](https://developer.android.com/reference/android/app/Instrumentation?hl=zh-cn) API。

**注意**：除非您的应用满足一组特定要求，否则 Google Play 会限制应用使用某些 API 和权限。详细了解这些政策：

- [使用 AccessibilityService API](https://support.google.com/googleplay/android-developer/answer/10964491?hl=zh-cn)
- [使用广泛的软件包（应用）可见性 (QUERY_ALL_PACKAGES) 权限](https://support.google.com/googleplay/android-developer/answer/10158779?hl=zh-cn)。

### 应用互动

用户如何与您的应用互动的相关信息。例如，他们对某个页面的访问次数或点按的内容。

### 应用内搜索记录

与用户在您的应用中搜索的内容相关的信息。

### 已安装的应用

与用户设备上安装的应用相关的信息。

### 其他由用户生成的内容

此处或任何其他部分未列出的其他用户生成的内容。例如，用户简介、备注或开放式回复。

### 其他操作

此处未列出的任何其他用户操作。例如，游戏内容信息、喜欢的内容或对话选项。

## 网页浏览

您的应用或应用所含库可以通过多种方式访问与网页浏览相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 向用户发送请求，以将您的应用设为默认浏览器应用。
- 保留浏览器缓存 Cookie。
- [创建搜索界面](https://developer.android.com/guide/topics/search/search-dialog?hl=zh-cn)。

## 应用信息和性能

您的应用或应用所含库可以通过多种方式访问与应用信息和性能相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 声明 [`BATTERY_STATS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BATTERY_STATS) 权限。
- 使用如下 API：
    - [`ActivityManager`](https://developer.android.com/reference/android/app/ActivityManager?hl=zh-cn)
    - [`ApplicationErrorReport`](https://developer.android.com/reference/android/app/ApplicationErrorReport?hl=zh-cn)
    - [`ApplicationExitInfo`](https://developer.android.com/reference/android/app/ApplicationExitInfo?hl=zh-cn)
    - [`ASurfaceControl](https://developer.android.com/ndk/reference/group/native-activity?hl=zh-cn#asurfacecontrol)
    - [`BatteryManager`](https://developer.android.com/reference/android/os/BatteryManager?hl=zh-cn)
    - [`Benchmark`](https://developer.android.com/studio/profile/benchmarking-overview?hl=zh-cn)
    - [`Choreographer`](https://developer.android.com/ndk/reference/group/choreographer?hl=zh-cn)
    - [`Debug`](https://developer.android.com/reference/android/os/Debug?hl=zh-cn)
    - [`HealthStats`](https://developer.android.com/reference/android/os/health/HealthStats?hl=zh-cn)
    - [`Macrobenchmark`](https://developer.android.com/studio/profile/macrobenchmark-intro?hl=zh-cn)
    - [`PowerManager`](https://developer.android.com/reference/android/os/PowerManager?hl=zh-cn)
    - [`StrictMode`](https://developer.android.com/reference/android/os/StrictMode?hl=zh-cn)
- 调用如下方法：
    - 来自 `AudioManager` API 的 [`getAudioDevicesForAttributes()`](https://developer.android.com/reference/android/media/AudioManager?hl=zh-cn#getAudioDevicesForAttributes\(android.media.AudioAttributes\)) 和 [`getDirectProfilesForAttributes()`](https://developer.android.com/reference/android/media/AudioManager?hl=zh-cn#getDirectProfilesForAttributes\(android.media.AudioAttributes\))。

### 崩溃日志

您的应用的崩溃日志数据，例如，应用的崩溃次数、堆栈轨迹或与崩溃直接相关的其他信息。

### 诊断

与您的应用性能相关的信息，例如电池续航时间、加载时间、延迟时间、帧速率或任何技术诊断信息。

### 其他应用性能数据

此处未列出的任何其他应用性能数据。

## 设备 ID 或其他 ID

此类 ID 的示例包括：[IMEI 识别码](https://developer.android.com/reference/android/telephony/TelephonyManager?hl=zh-cn#getImei\(int\))、[MAC 地址](https://developer.android.com/reference/android/net/wifi/WifiInfo?hl=zh-cn#getMacAddress\(\))、[Widevine 设备 ID](https://developers.google.com/widevine/drm/client/android/vendor-extensions?hl=zh-cn#bytearray_properties)、[Firebase 安装 ID](https://firebase.google.com/docs/projects/manage-installations?hl=zh-cn#retrieve_client_identifers) 或[广告标识符](https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient?hl=zh-cn)。

您的应用或应用所含库可以通过多种方式访问与设备或其他 ID 相关的用户数据。下面列出了几个示例，但并非详尽无遗：

- 至少声明以下其中一项权限：
    - [`AD_ID`](https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient.Info?hl=zh-cn#public-string-getid)（Google Play 服务权限）
    - `READ_PRIVILEGED_PHONE_STATE`
- 使用 [`AdvertisingIdClient`](https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient?hl=zh-cn) 等 API。
- 通过 IP 地址或接入点名称推导设备或其他标识符信息。

## 版本记录

下表对本页中的已变更内容进行了总结：

| 日期               | 变更说明                                                                                                                                                |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2022 年 12 月 13 日 | 更新了位置信息、健康与健身、照片和视频、应用信息和性能以及设备或其他 ID 类别，以提及 Android 13 中引入的功能。                                                                                     |
| 2022 年 2 月 24 日  | 将**设备或其他标识符**数据类型的类别名称更改为**设备 ID 或其他 ID**。更改了几种数据类型的名称和说明，包括**个人信息**、**财务信息**、**健康与健身**、**信息**和**应用活动**类别中的数据类型。                                    |
| 2022 年 1 月 4 日   | 更新了“性取向和性别认同”数据类型。此[数据类型现在仅指性取向](https://developer.android.com/privacy-and-security/declare-data-use?hl=zh-cn#sexual-orientation)。性别认同现属于其他个人信息的范畴。 |
| 2021 年 10 月 18 日 | 发布了初始版本。                                                                                                                                            |