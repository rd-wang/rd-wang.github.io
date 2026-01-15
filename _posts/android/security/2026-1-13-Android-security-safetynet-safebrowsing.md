---
title: Android-安全性-Safe Browsing API
date: 2026-1-13 09:39:59 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Safety-Net
description: 应用可以使用此 API 来确定某个特定网址是否已被 Google 归类为已知威胁。
math: true
---
SafetyNet Safe Browsing API 是一个由 [Google Play 服务](https://developers.google.com/android?hl=zh-cn)提供支持的库，提供的服务用于确定网址是否已被 Google 标记为已知威胁。

应用可以使用此 API 来确定某个特定网址是否已被 Google 归类为已知威胁。在内部，SafetyNet 为 Google 开发的 Safe Browsing Network Protocol v4 实现了一个客户端。客户端代码和 v4 网络协议都旨在保护用户的隐私，并将电池电量和带宽消耗量降至最低。您可以使用此 API 以最优化资源的方式在 Android 上充分利用 Google 安全浏览服务，且无需实现其网络协议。

本文档介绍了如何使用 SafetyNet Safe Browsing Lookup API 检查网址是否存在已知威胁。

## 服务条款

使用 Safe Browsing API 即表示您同意遵守《[服务条款](https://developers.google.com/safe-browsing/terms?hl=zh-cn)》。请先阅读并了解所有适用的条款及政策，然后再使用 Safe Browsing API。

## 请求并注册 Android API 密钥

在使用 Safe Browsing API 之前，请创建并注册 Android API 密钥。如需了解具体步骤，请参阅[安全浏览使用入门](https://developers.google.com/safe-browsing/v4/get-started?hl=zh-cn)页面。

## 添加 SafetyNet API 依赖项

在使用 Safe Browsing API 之前，请先将 SafetyNet API 添加到您的项目。如果您使用的是 Android Studio，请将此依赖项添加到您的应用级 Gradle 文件。如需了解详情，请参阅[使用 SafetyNet 抵御安全威胁](https://developer.android.com/training/safetynet?hl=zh-cn#before-you-begin)。

## 初始化 API

如需使用 Safe Browsing API，您必须先将其初始化，具体方法是调用 [`initSafeBrowsing()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetClient.html?hl=zh-cn#initSafeBrowsing\(\)) 并等待其完成。以下代码段提供了一个示例：

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#kotlin)
```kotlin
Tasks.await(SafetyNet.getClient(this).initSafeBrowsing())
```

[Java](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#java)
```java
Tasks.await(SafetyNet.getClient(this).initSafeBrowsing());
```


>**注意:** 为了尽量减少应用初始化的影响，请尽早在 activity 的 [`onResume()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onResume\(\)) 方法中调用 `initSafeBrowsing()`。

## 请求检查网址

您的应用可以进行网址检查，以确定某个网址是否属于已知威胁。某些威胁类型可能与您的具体应用无关。您可以通过该 API 视需要选择哪些威胁类型是重要的。您可以指定多种已知的威胁类型。

### 发送网址检查请求

该 API 与所使用的 scheme 无关，因此您可以传递含（或不含）scheme 的网址。例如：

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#kotlin)
```kotlin
var url = "https://www.google.com"
```
[Java](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#java)
```java
String url = "https://www.google.com";
```

和

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#kotlin)
```kotlin
var url = "www.google.com";
```
[Java](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#java)
```java
String url = "www.google.com"
```

都是有效的。

以下代码演示了如何发送网址检查请求：

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#kotlin)
```kotlin
SafetyNet.getClient(this).lookupUri(
       url,
       SAFE_BROWSING_API_KEY,
       SafeBrowsingThreat.TYPE_POTENTIALLY_HARMFUL_APPLICATION,
       SafeBrowsingThreat.TYPE_SOCIAL_ENGINEERING
)
       .addOnSuccessListener(this) { sbResponse ->
           // Indicates communication with the service was successful.
           // Identify any detected threats.
           if (sbResponse.detectedThreats.isEmpty()) {
               // No threats found.
           } else {
               // Threats found!
           }
       }
       .addOnFailureListener(this) { e: Exception ->
           if (e is ApiException) {
               // An error with the Google Play Services API contains some
               // additional details.
               Log.d(TAG, "Error: ${CommonStatusCodes.getStatusCodeString(e.statusCode)}")

               // Note: If the status code, s.statusCode,
               // is SafetyNetStatusCode.SAFE_BROWSING_API_NOT_INITIALIZED,
               // you need to call initSafeBrowsing(). It means either you
               // haven't called initSafeBrowsing() before or that it needs
               // to be called again due to an internal error.
           } else {
               // A different, unknown type of error occurred.
               Log.d(TAG, "Error: ${e.message}")
           }
       }
```

[Java](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#java)
```java

SafetyNet.getClient(this).lookupUri(url,
         SAFE_BROWSING_API_KEY,
         SafeBrowsingThreat.TYPE_POTENTIALLY_HARMFUL_APPLICATION,
         SafeBrowsingThreat.TYPE_SOCIAL_ENGINEERING)
   .addOnSuccessListener(this,
       new OnSuccessListener<SafetyNetApi.SafeBrowsingResponse>() {
           @Override
           public void onSuccess(SafetyNetApi.SafeBrowsingResponse sbResponse) {
               // Indicates communication with the service was successful.
               // Identify any detected threats.
               if (sbResponse.getDetectedThreats().isEmpty()) {
                   // No threats found.
               } else {
                   // Threats found!
               }
        }
   })
   .addOnFailureListener(this, new OnFailureListener() {
           @Override
           public void onFailure(@NonNull Exception e) {
               // An error occurred while communicating with the service.
               if (e instanceof ApiException) {
                   // An error with the Google Play Services API contains some
                   // additional details.
                   ApiException apiException = (ApiException) e;
                   Log.d(TAG, "Error: " + CommonStatusCodes
                       .getStatusCodeString(apiException.getStatusCode()));

                   // Note: If the status code, apiException.getStatusCode(),
                   // is SafetyNetStatusCode.SAFE_BROWSING_API_NOT_INITIALIZED,
                   // you need to call initSafeBrowsing(). It means either you
                   // haven't called initSafeBrowsing() before or that it needs
                   // to be called again due to an internal error.
               } else {
                   // A different, unknown type of error occurred.
                   Log.d(TAG, "Error: " + e.getMessage());
               }
           }
   });
```
### 读取网址检查响应

使用返回的 [`SafetyNetApi.SafeBrowsingResponse`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetApi.SafeBrowsingResponse?hl=zh-cn) 对象，调用其 [`getDetectedThreats()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetApi.SafeBrowsingResponse.html?hl=zh-cn#getDetectedThreats\(\)) 方法，该方法会返回 [`SafeBrowsingThreat`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafeBrowsingThreat?hl=zh-cn) 对象的列表。如果返回的列表为空，则表示 API 未检测到任何已知威胁。如果该列表不为空，则对列表中的每个元素都调用 [`getThreatType()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafeBrowsingThreat.html?hl=zh-cn#getThreatType\(\))，以确定 API 检测到了哪些已知威胁。

如需查看建议的警告语言，请参阅 [Safe Browsing API 开发者指南](https://developers.google.com/safe-browsing/v4/usage-limits?hl=zh-cn#suggested-warning-language)。

### 指定相关威胁类型

`SafeBrowsingThreat` 类中的常量包含当前支持的威胁类型：

|威胁类型|定义|
|---|---|
|`TYPE_POTENTIALLY_HARMFUL_APPLICATION`|这种威胁类型用于识别被标记为包含潜在有害应用的页面的网址。|
|`TYPE_SOCIAL_ENGINEERING`|这种威胁类型用于识别被标记为包含社会工程学威胁的页面的网址。|

使用该 API 时，您可以添加威胁类型常量作为参数。您可以根据应用的需要添加任意数量的威胁类型常量，但只能使用未被标记为已废弃的常量。

## 关闭安全浏览会话

如果应用无需长时间使用 Safe Browsing API，请检查应用中的所有必要网址，然后使用 [`shutdownSafeBrowsing()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetClient.html?hl=zh-cn#shutdownSafeBrowsing\(\)) 方法关闭安全浏览会话：

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#kotlin)
```kotlin
SafetyNet.getClient(this).shutdownSafeBrowsing()
```
[Java](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn#java)
```java
SafetyNet.getClient(this).shutdownSafeBrowsing();
```


建议在 activity 的 [`onPause()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onPause\(\)) 方法中调用 `shutdownSafeBrowsing()`，在 activity 的 `onResume()` 方法中调用 [`initSafeBrowsing()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetClient.html?hl=zh-cn#initSafeBrowsing\(\))。但是，在调用 [`lookupUri()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetClient.html?hl=zh-cn#lookupUri\(java.lang.String,%20java.lang.String,%20int...\)) 之前，请确保 `initSafeBrowsing()` 已执行完毕。确保您的会话始终处于最新状态，从而减少应用中的内部错误。

## SafetyNet Safe Browsing API 收集的数据

SafetyNet Safe Browsing API 在与 Android 上的安全浏览服务通信时会自动收集以下数据：

|流量|说明|
|---|---|
|应用活动|收集本地哈希前缀匹配之后的网址哈希前缀，用于检测恶意网址。|

虽然力求做到尽可能公开透明，但对于 [Google Play 的“数据安全”部分](https://support.google.com/googleplay/android-developer/answer/10787469?hl=zh-cn)要求您填写的关于应用的用户数据收集、分享和安全做法的表单，需自行负责决定如何回应。

