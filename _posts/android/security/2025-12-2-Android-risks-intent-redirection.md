---
title: Android-安全风险-intent重定向
date: 2025-12-2 11:30:00 +0800
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
description: 当在存在漏洞的应用上下文中启动新组件时，如果攻击者能够部分或完全控制 intent 内容，就会出现 intent 重定向问题。
math: true
---

# intent 重定向

**OWASP 类别：** [MASVS-PLATFORM：平台交互](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概述

当在存在漏洞的应用上下文中启动新组件时，如果攻击者能够部分或完全控制 intent 内容，就会出现 intent 重定向问题。

用于启动新组件的 intent 可以通过多种方式提供，最常见的方式是将其作为字段中的序列化 intent `extras`，或者将其编组为字符串并进行解析。对参数进行部分控制也可以达到同样的效果。

## 影响

影响程度可能有所不同。攻击者可能会在存在漏洞的应用中执行内部功能，或者访问私有组件，例如未导出的 ContentProvider 对象。

## 缓解措施

通常情况下，不要公开与重定向嵌套 intent 相关的功能。如果无法避免，请采取以下缓解措施：

- 对打包信息进行适当的清理。务必记住检查或清除标志（`FLAG_GRANT_READ_URI_PERMISSION, FLAG_GRANT_WRITE_URI_PERMISSION, FLAG_GRANT_PERSISTABLE_URI_PERMISSION, and FLAG_GRANT_PREFIX_URI_PERMISSION`），并检查 intent 重定向到哪里。[`IntentSanitizer`](https://developer.android.com/reference/kotlin/androidx/core/content/IntentSanitizer)可以帮助完成此过程。
- 使用[`PendingIntent`](https://developer.android.com/guide/components/intents-filters#PendingIntent)对象。这可以防止组件被导出，并使目标操作 intent 不可变。

应用程序可以使用 [`ResolveActivity`](https://developer.android.com/reference/android/content/Intent#resolveActivity\(android.content.pm.PackageManager\))方法检查 intent 的重定向位置：

[Kotlin](https://developer.android.com/privacy-and-security/risks/intent-redirection#kotlin)
```kotlin
val intent = getIntent()
// Get the component name of the nested intent.
val forward = intent.getParcelableExtra<Parcelable>("key") as Intent
val name: ComponentName = forward.resolveActivity(packageManager)
// Check that the package name and class name contain the expected values.
if (name.packagename == "safe_package" && name.className == "safe_class") {
    // Redirect the nested intent.
    startActivity(forward)
}
```
[Java](https://developer.android.com/privacy-and-security/risks/intent-redirection#java)
```java
Intent intent = getIntent()
// Get the component name of the nested intent.
Intent forward = (Intent) intent.getParcelableExtra("key");
ComponentName name = forward.resolveActivity(getPackageManager());
// Check that the package name and class name contain the expected values.
if (name.getPackageName().equals("safe_package") &&
        name.getClassName().equals("safe_class")) {
    // Redirect the nested intent.
    startActivity(forward);
}
```

应用程序可以使用[`IntentSanitizer`](https://developer.android.com/reference/kotlin/androidx/core/content/IntentSanitizer)类似于以下逻辑：

[Kotlin](https://developer.android.com/privacy-and-security/risks/intent-redirection#kotlin)
```kotlin
val intent = IntentSanitizer.Builder()
     .allowComponent("com.example.ActivityA")
     .allowData("com.example")
     .allowType("text/plain")
     .build()
     .sanitizeByThrowing(intent)
```
[Java](https://developer.android.com/privacy-and-security/risks/intent-redirection#java)

```java
Intent intent = new  IntentSanitizer.Builder()
     .allowComponent("com.example.ActivityA")
     .allowData("com.example")
     .allowType("text/plain")
     .build()
     .sanitizeByThrowing(intent);
```

#### 默认保护

Android 16 引入了一项默认的安全加固方案，用于`Intent` 防御重定向漏洞。在大多数情况下，使用 Intent 的应用通常不会遇到任何兼容性问题。

##### 选择退出 intent 重定向处理

Android 16 引入了一个新的 API，允许应用选择退出启动安全保护。在某些情况下，默认的安全行为可能会干扰应用的正常使用，因此这种做法可能是必要的。

>**重要提示：**选择退出安全保护措施应谨慎行事，仅在绝对必要时才可这样做，因为这可能会增加安全漏洞的风险。在使用此 API 之前，请仔细评估其对应用安全性的潜在影响。

在 Android 16 中，您可以使用 `Intent`对象上的`removeLaunchSecurityProtection()`方法来选择退出安全保护。例如：

```kotlin
val i = intent
val iSublevel: Intent? = i.getParcelableExtra("sub_intent")
iSublevel?.removeLaunchSecurityProtection() // Opt out from hardening
iSublevel?.let { startActivity(it) }
```

#### 常见错误

- 检查`getCallingActivity()`是否返回非空值。恶意应用程序可能会为此函数提供空值。
- 假设`checkCallingPermission()`在所有上下文中都能正常工作，或者当该方法实际返回整数时会抛出异常。

#### 调试功能

对于面向 Android 12（API 级别 31）或更高版本的应用，可以启用 [调试功能](https://developer.android.com/guide/components/intents-filters#DetectUnsafeIntentLaunches)，在某些情况下，该功能可以帮助检测应用是否正在执行不安全的 intent 启动。

如果您的应用执行以下**两个**操作，系统将检测到不安全的 intent 启动，并`StrictMode`发生违规：

- 您的应用会将嵌套的 intent 从已传递 intent 的 extra 中解包出来。
- 您的应用会立即使用该嵌套的 intent 启动一个应用组件，例如将 intent 传递给`startActivity()`、`startService()`或`bindService()`。

## 资源

- [ intent 重定向的补救措施](https://support.google.com/faqs/answer/9267555)
- [ intent 和 intent 过滤器](https://developer.android.com/guide/components/intents-filters#DetectUnsafeIntentLaunches)

