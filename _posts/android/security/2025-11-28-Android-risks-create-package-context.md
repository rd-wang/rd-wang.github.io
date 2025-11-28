---
title: Android-安全风险-创建上下文
date: 2025-11-28 15:34:53 +0800
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
description: 如果应用以不安全的方式使用 createPackageContext，可能会导致恶意应用能够在易受攻击的应用环境中执行任意代码。
math: true
---

# 创建上下文

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

```java
public abstract Context createPackageContext (String packageName, int flags)
```

当开发者想要在自己的应用中为其他应用创建上下文时，可以使用方法 [`createPackageContext`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\))。

例如，如果开发者想要从第三方应用获取资源或调用其中的方法，则需要使用 `createPackageContext`。

不过，如果应用使用 [`CONTEXT_IGNORE_SECURITY`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_IGNORE_SECURITY) 和 [`CONTEXT_INCLUDE_CODE`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_INCLUDE_CODE) 标志调用 `createPackageContext`，然后调用 [`getClassLoader()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getClassLoader\(\))，则可能会导致应用容易受到恶意应用执行代码的影响。例如，当攻击者冒充开发者预计在用户设备上存在的未声明软件包名称（软件包占用）时，就可能会发生这种情况。

总而言之，应用必须满足以下条件，才能受到此类攻击：

易受攻击的应用：

- 使用 [`CONTEXT_IGNORE_SECURITY`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_IGNORE_SECURITY) 和 [`CONTEXT_INCLUDE_CODE`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_INCLUDE_CODE) 调用 [`createPackageContext`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\))。
- 对检索到的上下文调用 [`getClassLoader()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getClassLoader\(\))。

恶意应用：

- 能够声明易受攻击的应用传递给 [`createPackageContext`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\)) 的软件包名称。
- 导出 android:appComponentFactory。

## 影响

如果应用以不安全的方式使用 createPackageContext，可能会导致恶意应用能够在易受攻击的应用环境中执行任意代码。

## 缓解措施

除非绝对必要，否则请勿使用 [`CONTEXT_IGNORE_SECURITY`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_IGNORE_SECURITY) 和 [`CONTEXT_INCLUDE_CODE`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#CONTEXT_INCLUDE_CODE) 调用 [`createPackageContext`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\))。

如果无法避免这种情况，请务必实现一种机制来验证您要针对哪个软件包执行 [`createPackageContext`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\))（例如，通过验证软件包的签名）。

## 资源

- [createPackageContext 文档](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createPackageContext\(java.lang.String,%20int\))
- [OverSecured 博文（关于 createPackageContext 代码执行）](https://blog.oversecured.com/Android-arbitrary-code-execution-via-third-party-package-contexts)
