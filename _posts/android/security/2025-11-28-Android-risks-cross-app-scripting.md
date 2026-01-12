---
title: Android-安全性-安全风险-跨应用脚本
date: 2025-11-28 11:31:20 +0800
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
description: 跨应用脚本攻击与在受害应用环境中执行恶意代码
math: true
---
# 跨应用脚本

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

WebView 是 Android 应用中的一种嵌入式浏览器组件， 便于在应用中显示 Web 内容。它可以呈现 HTML、CSS 和 应用界面中的 JavaScript

跨应用脚本攻击与在受害应用环境中执行恶意代码密切相关。在本文档中，我们将仅讨论将恶意 JavaScript 代码注入易受攻击的 WebView 的情况。

如果应用在没有进行充分验证或净化的情况下将恶意 JavaScript 接受到 WebView 中，则该应用容易受到跨应用脚本攻击。

## 影响

如果攻击者控制的 JavaScript 内容未经验证或清理就传递给易受攻击的应用的 WebView，攻击者便可以利用跨应用脚本漏洞。因此，攻击者提供的 JavaScript 代码会在受害应用的 WebView 上下文中执行。然后，恶意 JavaScript 代码可以使用与受害应用相同的权限，这可能会导致敏感用户数据被盗和账号被盗用。

## 缓解措施

### 停用 JavaScript

如果您的应用不需要 JavaScript，请停用它，以确保它不会成为威胁：

[Kotlin](https://developer.android.com/privacy-and-security/risks/cross-app-scripting?hl=zh-cn#kotlin)

```kotlin
// Get the WebView Object
val webView = findViewById<WebView>(R.id.webView)
val webSettings = webView.settings

// Disable JavaScript
webSettings.javaScriptEnabled = false
```

[Java](https://developer.android.com/privacy-and-security/risks/cross-app-scripting?hl=zh-cn#java)

```java
// Get the WebView Object
WebView webView = (WebView) findViewById(R.id.webView);
WebSettings webSettings = webView.getSettings();

// Disable JavaScript for the WebView
webSettings.setJavaScriptEnabled(false);
```

如果您的应用确实需要 JavaScript，请确保您拥有或控制传递给 WebView 的所有 JavaScript。避免允许 WebView 任意执行 JavaScript，请参阅下一部分中的指南。

### 确保仅将预期内容加载到 WebView

使用 [`shouldOverrideUrlLoading()`](https://developer.android.com/reference/android/webkit/WebViewClient?hl=zh-cn#shouldOverrideUrlLoading\(android.webkit.WebView,%20android.webkit.WebResourceRequest\))、[`loadUrl()`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#loadUrl\(java.lang.String\)) 或 [`evaluateJavascript()`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#evaluateJavascript\(java.lang.String,%20android.webkit.ValueCallback%3Cjava.lang.String%3E\))`,` 等方法时，请确保对传递给它们的所有网址进行检查。如前所述，传递给 WebView 的任何 JavaScript 都应 来自预期的域，因此请务必验证正在加载的内容。

如需获取实用建议和示例，请参阅 OWASP 的输入验证[文档](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)以及适用于 WebView 的此 Android 安全[核对清单](https://blog.oversecured.com/Android-security-checklist-webview/)。

### 为 WebView 设置安全文件访问权限设置

确保文件无法访问可以阻止任意 JavaScript： 以下 [`WebSettings`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn) 应 在保障文件访问安全时考虑以下因素：

- 停用文件访问权限。默认情况下，系统会将 [`setAllowFileAccess`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowFileAccess\(boolean\)) 设为 `True` API 级别 29 及更低级别（将允许访问本地文件）。在 API 级别 30 及更高级别中，默认值为 `False`。为确保不允许访问文件，请将 `setAllowFileAccess` 明确设置为 `False`
- 停用内容访问权限。[`setAllowContentAccess`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowContentAccess\(boolean\)) 的默认设置为 `True`。通过内容网址访问权限，WebView 可以从内容加载内容 提供程序。如果您的应用不需要内容访问权限， 将 `setAllowContentAccess` 设为 `False` 以防发生潜在的滥用行为， 跨应用脚本攻击
    
- kotlin `kotlin webView.settings.javaScriptEnabled = false webView.settings.domStorageEnabled = true webView.settings.allowFileAccess = false webView.settings.allowContentAccess = false`
    
- Java `java webView.getSettings().setJavaScriptEnabled(false); webView.getSettings().setDomStorageEnabled(true); webView.getSettings().setAllowFileAccess(false); webView.getSettings().setAllowContentAccess(false);`
    

### 启用安全浏览功能

在 [`AndroidManifest.xml`](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=zh-cn) 中启用安全浏览功能，以扫描传递给 钓鱼式攻击或恶意网域的 WebView：

```xml
<meta-data android:name="android.webkit.WebView.EnableSafeBrowsing"
   android:value="true" />
```

## 资源

- [“安全浏览”文档](https://developer.android.com/privacy-and-security/safetynet/safebrowsing?hl=zh-cn)
- [WebView 开发者参考文档](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn)
- [适用于 WebView 的 WebSettings 开发者参考文档](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn)
- [setAllowFileAccess 开发者文档](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowFileAccess\(boolean\))
- [setAllowContentAccess 开发者参考文档](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowContentAccess\(boolean\))
