---
title: Android-安全风险-隐式 intent 盗用
date: 2025-11-28 16:04:52 +0800
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
description: 通过盗用隐式 intent，攻击者可以读取或修改 intent 的内容，并拦截 intent 以执行操作。这会导致敏感信息/数据泄露或启用由攻击者控制的组件。
math: true
---


# 隐式 intent 盗用

**OWASP 类别：** [MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

如果应用在调用 intent 时未指定完全限定的组件类名称或软件包，便会出现[隐式 intent](https://developer.android.com/guide/components/intents-filters?hl=zh-cn#Types) 盗用漏洞。这会让恶意应用取代预期的应用，通过注册 intent 过滤器来拦截 intent。

根据 intent 内容，攻击者可能会读取敏感信息或与可变对象互动，例如[可变](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn#FLAG_MUTABLE) [PendingIntent](https://developer.android.com/reference/android/app/PendingIntent?hl=zh-cn)或 Binder。

## 影响

通过盗用隐式 intent，攻击者可以读取或修改 intent 的内容，并拦截 intent 以执行操作。这会导致敏感信息/数据泄露或启用由攻击者控制的组件。

## 缓解措施

通过调用 `setPackage()`，将 intent 设为显式 intent，如以下代码段所示：

[Kotlin](https://developer.android.com/privacy-and-security/risks/implicit-intent-hijacking?hl=zh-cn#kotlin)
```kotlin
val intent = Intent("android.intent.action.CREATE_DOCUMENT").apply {
    addCategory("android.intent.category.OPENABLE")
    setPackage("com.some.packagename")
    setType("*/*")
    putExtra("android.intent.extra.LOCAL_ONLY", true)
    putExtra("android.intent.extra.TITLE", "Some Title")
}

startActivity(intent)
```
[Java](https://developer.android.com/privacy-and-security/risks/implicit-intent-hijacking?hl=zh-cn#java)

```java
Intent intent = new Intent("android.intent.action.CREATE_DOCUMENT");
intent.addCategory("android.intent.category.OPENABLE");

intent.setPackage("com.some.packagename");

intent.setType("*/*");
intent.putExtra("android.intent.extra.LOCAL_ONLY", true);
intent.putExtra("android.intent.extra.TITLE", "Some Title");
startActivity(intent);
```

如果您需要使用隐式 intent，请忽略您不想公开的任何敏感信息或可变对象。当应用无法确切了解哪个应用将执行操作（例如，写电子邮件、拍照等）时，可能需要使用隐式 intent。

## 资源

- [清单 intent 过滤器元素](https://developer.android.com/guide/topics/manifest/intent-filter-element?hl=zh-cn)
    
- [特许权限许可名单](https://source.android.com/devices/tech/config/perms-allowlist?hl=zh-cn)
    
- [intent 和 intent 过滤器](https://developer.android.com/guide/components/intents-filters?hl=zh-cn)
    
- [ContentResolver PROTECTED_ACTIONS](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/services/core/java/com/android/server/pm/ComponentResolver.java;l=95?q=PROTECTED_ACTIONS&hl=zh-cn)
    
- [为隐式 intent 强制使用选择器](https://developer.android.com/guide/components/intents-%0Afilters?hl=zh-cn#ForceChooser)
    
- [常见隐式 intent](https://developer.android.com/guide/components/intents-common?hl=zh-cn)
