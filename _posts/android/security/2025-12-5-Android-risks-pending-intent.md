---
title: Android-安全性-安全风险-pending-intent
date: 2025-12-5 15:58:41 +0800
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
description: PendingIntent是指向系统维护的令牌的引用。应用程序 A 可以向应用程序 B 传递 PendingIntent，以便允许应用程序 B 代表应用程序 A 执行预定义的操作；无论应用程序 A 是否仍然运行。
math: true
---
# 待处理 intent

**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

[`PendingIntent`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn) 是对系统维护的令牌的引用。应用 A 可以将 PendingIntent 传递到应用 B，以允许应用 B 代表应用 A 执行预定义的操作，无论应用 A 是否仍在运行。

## 风险：可变的待处理 intent

PendingIntent 是可变的，这意味着应用 B 可以按照 [`fillIn()`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#fillIn\(android.content.Intent,%20int\)) 文档中所述的逻辑更新用于指定操作的内部 intent。换言之，恶意应用可能会修改 PendingIntent 的某些未填充字段，从而允许攻击者访问存在漏洞的应用中原本不支持导出的组件。

### 影响

该漏洞的影响程度取决于应用程序未导出目标功能的实现方式。

### 缓解措施

#### 一般措施

请确保操作、组件和包设置正确，以避免最严重的漏洞：

[Kotlin](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#kotlin)

```kotlin
val intent = Intent(intentAction)

// Or other component setting APIs e.g. setComponent, setClass
intent.setClassName(packageName, className)

PendingIntent pendingIntent =
    PendingIntent.getActivity(
        context,
        /* requestCode = */ 0,
        intent, /* flags = */ PendingIntent.FLAG_IMMUTABLE
    )

```
[Java](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#java)

```java
Intent intent = new Intent(intentAction);

// Or other component setting APIs e.g. setComponent, setClass
intent.setClassName(packageName, className);

PendingIntent pendingIntent =
        PendingIntent.getActivity(
            getContext(),
            /* requestCode= */ 0,
            intent, /* flags= */ 0);

```

#### 标记 IMMUTABLE

如果您的应用以 Android 6（API 级别 23）或更高版本为目标平台，请[指定可变性](https://developer.android.com/guide/components/intents-filters?hl=zh-cn#DeclareMutabilityPendingIntent)。例如，可以通过使用 [`FLAG_IMMUTABLE`](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#FLAG_IMMUTABLE) 来防止恶意应用填充未填充的字段：

[Kotlin](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#kotlin)
```kotlin
val pendingIntent =
    PendingIntent.getActivity(
        context,
        /* requestCode = */ 0,
        Intent(intentAction),
        PendingIntent.FLAG_IMMUTABLE)
```
[Java](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#java)

```java
PendingIntent pendingIntent =
        PendingIntent.getActivity(
            getContext(),
            /* requestCode= */ 0,
            new Intent(intentAction),
            PendingIntent.FLAG_IMMUTABLE);
```

在 Android 11（API 级别 30）及更高版本中，您必须指定要将哪些字段设置为可变字段，以缓解此类意外漏洞。

### 资源

- [修复 PendingIntent 漏洞](https://support.google.com/faqs/answer/9267555?hl=zh-cn)
    
- [有关此漏洞的博文](https://valsamaras.medium.com/pending-intents-a-pentesters-view-92f305960f03)
    

---

## 风险：重放待处理 intent

除非设置了 [FLAG_ONE_SHOT](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#FLAG_ONE_SHOT) 标志，否则 PendingIntent 可以重放。请务必使用 FLAG_ONE_SHOT，以避免重放攻击（执行不应重复的操作）。

### 影响

此漏洞的影响取决于意图接收端的具体实现方式。恶意应用程序可以利用未设置 FLAG_ONE_SHOT 标志而创建的 PendingIntent，捕获并重用该意图，从而重复执行原本只能执行一次的操作。

### 缓解措施

不应多次触发的待处理 intent 应使用 [FLAG_ONE_SHOT](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#FLAG_ONE_SHOT) 标志来避免重放攻击。

[Kotlin](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#kotlin)
```kotlin
val pendingIntent =
      PendingIntent.getActivity(
          context,
          /* requestCode = */ 0,
          Intent(intentAction),
          PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_ONE_SHOT)
```
[Java](https://developer.android.com/privacy-and-security/risks/pending-intent?hl=zh-cn#java)

```java
PendingIntent pendingIntent =
        PendingIntent.getActivity(
            getContext(),
            /* requestCode= */ 0,
            new Intent(intentAction),
            PendingIntent.FLAG_IMMUTABLE | PendingIntent.FLAG_ONE_SHOT);
```

### 资源

- [PendingIntent.FLAG_ONE_SHOT](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#FLAG_ONE_SHOT)

---

## 资源

- [PendingIntent 文档](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn)
    
- [PendingIntent 和 intent 过滤器](https://developer.android.com/guide/components/intents-filters?hl=zh-cn#PendingIntent)

