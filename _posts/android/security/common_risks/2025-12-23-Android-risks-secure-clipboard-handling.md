---
title: Android-安全性-安全风险-安全的剪贴板处理
date: 2025-12-23 11:38:14 +0800
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
description: 利用剪贴板操作不当可能导致用户敏感信息或财务数据被恶意攻击者窃取。这可能有助于攻击者实施进一步的攻击，例如网络钓鱼或身份盗窃。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

Android 提供了一个名为[剪贴板](https://developer.android.com/develop/ui/views/touch-and-input/copy-paste?hl=zh-cn#Clipboard)的强大框架，用于在应用程序之间复制和粘贴数据。如果此功能使用不当，可能会将用户数据暴露给未经授权的恶意行为者或应用程序。

剪贴板数据泄露的具体风险取决于应用程序的性质及其处理的个人身份信息 (PII)。对金融应用程序的影响尤其严重，因为它们可能会泄露支付数据，或者对处理双因素身份验证 (2FA) 代码的应用程序的影响也很大。

可用于窃取剪贴板数据的攻击途径因 Android 版本而异：

- [低于 Android 10（API 级别 29）的 Android 版本](https://developer.android.com/about/versions/10/privacy/changes?hl=zh-cn#clipboard-data)允许后台应用程序访问前台应用程序剪贴板信息，这可能会使恶意行为者直接访问任何复制的数据。
- 从 Android 12（API 级别 31）开始，每次应用程序访问剪贴板中的数据并粘贴时，都会向用户显示一条提示信息，这使得攻击更难不被察觉。此外，为了保护个人身份信息 (PII)，Android 支持`ClipDescription.EXTRA_IS_SENSITIVE` 或 `android.content.extra.IS_SENSITIVE` 特殊标志。这样一来，开发者就可以在键盘 GUI 中对剪贴板内容预览进行视觉混淆，防止复制的数据以明文形式显示，从而避免被恶意应用程序窃取。事实上，如果未实现上述任一标志，攻击者可能会通过偷窥或通过在后台运行时截取合法用户活动的屏幕截图或录制视频的恶意应用，窃取复制到剪贴板的敏感数据。

## 影响

利用剪贴板操作不当可能导致用户敏感信息或财务数据被恶意攻击者窃取。这可能有助于攻击者实施进一步的攻击，例如网络钓鱼或身份盗窃。
## 缓解措施

### 标记敏感数据

该解决方案用于在键盘 GUI 中对剪贴板内容预览进行视觉模糊处理。任何可以复制的敏感数据，例如密码或信用卡数据，都应该在调用 [`ClipboardManager.setPrimaryClip()`](https://developer.android.com/reference/android/content/ClipboardManager?hl=zh-cn#setPrimaryClip\(android.content.ClipData\)) 之前，先使用 `ClipDescription.EXTRA_IS_SENSITIVE` 或 `android.content.extra.IS_SENSITIVE` 进行标记

[Kotlin](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#kotlin)

```kotlin
// If your app is compiled with the API level 33 SDK or higher.
clipData.apply {
    description.extras = PersistableBundle().apply {
        putBoolean(ClipDescription.EXTRA_IS_SENSITIVE, true)
    }
}

// If your app is compiled with API level 32 SDK or lower.
clipData.apply {
    description.extras = PersistableBundle().apply {
        putBoolean("android.content.extra.IS_SENSITIVE", true)
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#java)

```java
// If your app is compiled with the API level 33 SDK or higher.
PersistableBundle extras = new PersistableBundle();
extras.putBoolean(ClipDescription.EXTRA_IS_SENSITIVE, true);
clipData.getDescription().setExtras(extras);

// If your app is compiled with API level 32 SDK or lower.
PersistableBundle extras = new PersistableBundle();
extras.putBoolean("android.content.extra.IS_SENSITIVE", true);
clipData.getDescription().setExtras(extras);
```
### 强制要求使用最新的 Android 版本

强制应用在 Android 10（API 29）或更高版本上运行，可以防止后台进程在前台应用中访问剪贴板数据。

要强制应用仅在 Android 10 (API 29) 或更高版本上运行，请在 Android Studio 的项目的 Gradle 构建文件中为版本设置设置以下值。

[Groovy](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#groovy)

```groovy
android {
      namespace = "com.example.testapp"
      compileSdk = [SDK_LATEST_VERSION]

      defaultConfig {
          applicationId = "com.example.testapp"
          minSdk = 29
          targetSdk = [SDK_LATEST_VERSION]
          versionCode = 1
          versionName = "1.0"
          ...
      }
      ...
    }
    ...
```
[Kotlin](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#kotlin)
```kotlin
android {
      namespace = "com.example.testapp"
      compileSdk = [SDK_LATEST_VERSION]

      defaultConfig {
          applicationId = "com.example.testapp"
          minSdk = 29
          targetSdk = [SDK_LATEST_VERSION]
          versionCode = 1
          versionName = "1.0"
          ...
      }
      ...
    }
    ...
```
### 在指定时间段后删除剪贴板内容

如果应用要在低于 Android 10（API 级别 29）的 Android 版本上运行，则任何后台应用都可以访问剪贴板数据。为了降低这种风险，可以实现一个功能，在特定时间段后清除复制到剪贴板的所有数据。[从 Android 13（API 级别 33）开始，系统会自动执行此函数](https://blog.google/products/android/android-13/?hl=zh-cn)。对于较旧的 Android 版本，可以通过在应用程序代码中包含以下代码片段来执行此删除操作。

[Kotlin](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#kotlin)

```kotlin
//The Executor makes this task Asynchronous so that the UI continues being responsive
backgroundExecutor.schedule({
    //Creates a clip object with the content of the Clipboard
    val clipboard = getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager
    val clip = clipboard.primaryClip
    //If SDK version is higher or equal to 28, it deletes Clipboard data with clearPrimaryClip()
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        clipboard.clearPrimaryClip()
    } else if (Build.VERSION.SDK_INT < Build.VERSION_CODES.P) {
    //If SDK version is lower than 28, it will replace Clipboard content with an empty value
        val newEmptyClip = ClipData.newPlainText("EmptyClipContent", "")
        clipboard.setPrimaryClip(newEmptyClip)
     }
//The delay after which the Clipboard is cleared, measured in seconds
}, 5, TimeUnit.SECONDS)
```
[Java](https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling?hl=zh-cn#java)
```java
//The Executor makes this task Asynchronous so that the UI continues being responsive

ScheduledExecutorService backgroundExecutor = Executors.newSingleThreadScheduledExecutor();

backgroundExecutor.schedule(new Runnable() {
    @Override
    public void run() {
        //Creates a clip object with the content of the Clipboard
        ClipboardManager clipboard = (ClipboardManager)getSystemService(Context.CLIPBOARD_SERVICE);
        ClipData clip = clipboard.getPrimaryClip();
        //If SDK version is higher or equal to 28, it deletes Clipboard data with clearPrimaryClip()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
            clipboard.clearPrimaryClip();
            //If SDK version is lower than 28, it will replace Clipboard content with an empty value
        } else if (Build.VERSION.SDK_INT < Build.VERSION_CODES.P) {
            ClipData newEmptyClip = ClipData.newPlainText("EmptyClipContent", "");
            clipboard.setPrimaryClip(newEmptyClip);
        }
    //The delay after which the Clipboard is cleared, measured in seconds
    }, 5, TimeUnit.SECONDS);
```
## 资源

- [剪贴板框架](https://developer.android.com/develop/ui/views/touch-and-input/copy-paste?hl=zh-cn#Clipboard)
- [在应用访问剪贴板数据时显示系统通知](https://developer.android.com/develop/ui/views/touch-and-input/copy-paste?hl=zh-cn#PastingSystemNotifications)
- [将敏感内容添加到剪贴板](https://developer.android.com/develop/ui/views/touch-and-input/copy-paste?hl=zh-cn#SensitiveContent)
- [Android 10 中的隐私权变更](https://developer.android.com/about/versions/10/privacy/changes?hl=zh-cn)
- [设置应用版本信息](https://developer.android.com/studio/publish/versioning?hl=zh-cn#appversioning)

