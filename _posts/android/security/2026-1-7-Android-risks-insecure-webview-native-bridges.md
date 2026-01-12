---
title: Android-安全性-安全风险-WebView-原生桥
date: 2026-1-7 11:36:52+0800
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
description: 恶意行为者可以利用 addJavascriptInterface、postWebMessage 和 postMessage 方法以访问、操纵或向 WebView 注入他们控制的代码。
math: true
---

**OWASP 类别：**[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

原生桥梁有时也称为 JavaScript 桥接器，是一种促进 WebView 和原生 Android 代码之间通信的机制。方法是使用 [`addJavascriptInterface`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#addJavascriptInterface\(java.lang.Object,%20java.lang.String\)) 方法。这样就可以在 WebView 中运行的 JavaScript 代码和 Android 应用程序的 Java 代码之间进行双向通信。`addJavascriptInterface` 方法将一个 Java 对象暴露给 WebView 的所有框架，任何框架都可以访问该对象名称并调用其方法。然而，应用程序没有机制来验证 WebView 中调用框架的来源，这引发了安全问题，因为内容的可靠性仍然无法确定。

原生桥也可以实现 HTML 消息通道，方法是使用 Android 的 [`WebViewCompat.postWebMessage`](https://developer.android.com/reference/androidx/webkit/WebViewCompat?hl=zh-cn#postWebMessage\(android.webkit.WebView,androidx.webkit.WebMessageCompat,android.net.Uri\)) 或 [`WebMessagePort.postMessage`](https://developer.android.com/reference/android/webkit/WebMessagePort?hl=zh-cn#postMessage\(android.webkit.WebMessage\))，以便与 JavaScript 进行通信 [`Window.postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)。`WebViewCompat.postWebMessage`和 `WebMessagePort.postMessage` 可接受通过 `Window.postMessage`（将在 WebView 中执行）。

与原生桥接相关的风险有多个：

- 基于 JavaScriptInterface 的桥： 
	- `addJavascriptInterface`  方法会将提供的 Java 对象注入到 WebView 的每个帧（包括 iframe）中，这意味着它容易受到恶意第三方的攻击，这些第三方会将帧注入到合法的网站中。 目标 API 级别 16 或更早版本的应用程序尤其容易受到攻击，因为这种方法可以允许 JavaScript 控制宿主应用程序。
    - 在原生启用桥接功能的 WebView 中反映不受信任的用户提供的内容，会导致跨站脚本攻击 (XSS)。
- 基于 MessageChannel 的桥： 
	- 由于消息通道端点缺乏来源检查，因此会接受来自任何发件人的消息，包括包含恶意代码的消息。
    - 可能意外地将 Java 暴露给任意 JavaScript。

## 影响

恶意行为者可以利用`addJavascriptInterface`、`postWebMessage` 和 `postMessage` 方法以访问、操纵或向 WebView 注入他们控制的代码。这可能导致用户被重定向到恶意网站、加载恶意内容，或者在其设备上运行恶意代码，从而提取敏感数据或实现权限提升。

## 风险：addJavascriptInterface 风险

WebView实现了浏览器的基本功能，例如页面渲染、导航和JavaScript执行。WebView可以用于应用程序内部，将网页内容显示为活动布局的一部分。使用 addJavascriptInterface 方法在 WebView 中实现原生桥接可能会产生安全问题，例如跨站脚本攻击 (XSS)，或者允许攻击者通过接口注入加载不受信任的内容，并以非预期的方式操纵宿主应用程序，以宿主应用程序的权限执行 Java 代码。

### 缓解措施

#### 停用 JavaScript

在 WebView 不需要 JavaScript 的情况下，请勿在 [`WebSettings`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn) 中调用 [`setJavaScriptEnabled`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setJavaScriptEnabled\(boolean\))（例如，在显示静态 HTML 内容时）。默认情况下，WebView 中 JavaScript 执行功能处于停用状态。

#### 在加载不受信任的内容时移除 JavaScript 接口

确保通过调用 JavaScript 接口中的对象来删除 [`removeJavascriptInterface`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#removeJavascriptInterface\(java.lang.String\))，然后 WebView。例如，您可以在调用 [`shouldInterceptRequest`](https://developer.android.com/reference/android/webkit/WebViewClient?hl=zh-cn#shouldInterceptRequest\(android.webkit.WebView,%20android.webkit.WebResourceRequest\)) 时执行此操作。

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges?hl=zh-cn#kotlin)
```kotlin
webView.removeJavascriptInterface("myObject")
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges?hl=zh-cn#java)

```java
webView.removeJavascriptInterface("myObject");
```

#### 仅通过 HTTPS 加载网络内容

如果您需要加载不受信任的内容，请确保 WebView 通过加密连接加载 Web 内容（另请参阅我们关于[明文通信](https://developer.android.com/privacy-and-security/risks/cleartext-communications?hl=zh-cn)的准则）。通过 `AndroidManifest` 文件，将 `android:usesCleartextTraffic` 设置为 `false`，或在[网络安全配置中禁止 HTTP 流量](https://developer.android.com/training/articles/security-config?hl=zh-cn)，来防止在未加密的连接上执行初始页面加载。。如需了解详情，请参阅 [`usesCleartextTraffic`](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#usesCleartextTraffic) 文档。

[Xml](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges?hl=zh-cn#xml)

```xml
<application
    android:usesCleartextTraffic="false">
    <!-- Other application elements -->
</application>
```

为确保重定向和后续应用浏览不会发生在未加密流量上，请检查 [`loadUrl`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#loadUrl\(java.lang.String\))  或 `shouldInterceptRequest`中的 HTTP 协议:

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges?hl=zh-cn#kotlin)
```kotlin
fun loadSecureUrl(webView: WebView?, url: String?) {
    webView?.let { wv ->  // Ensure valid WebView and URL
        url?.let {
            try {
                val uri = URI(url)
                if (uri.scheme.equals("https", ignoreCase = true)) { // Enforce HTTPS scheme for security
                    wv.loadUrl(url)
                } else {
                    // Log an error or handle the case where the URL is not secure
                    System.err.println("Attempted to load a non-HTTPS URL: $url")
                }
            } catch (e: Exception) {
                // Handle exception for improper URL format
                System.err.println("Invalid URL syntax: $url")
            }
        }
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges?hl=zh-cn#java)

```java
public void loadSecureUrl(WebView webView, String url) {
    if (webView != null && url != null) { // Ensure valid WebView and URL
        try {
            URI uri = new URI(url);
            String scheme = uri.getScheme();
            if ("https".equalsIgnoreCase(scheme)) { // Enforce HTTPS scheme for security
                webView.loadUrl(url);
            } else {
                // Log an error or handle the case where the URL is not secure
                System.err.println("Attempted to load a non-HTTPS URL: " + url);
            }
        } catch (URISyntaxException e) {
            // Handle exception for improper URL format
            System.err.println("Invalid URL syntax: " + url);
        }
    }
}
```

#### 验证不受信任的内容

如果 WebView 中加载了任何外部链接，则需要验证协议和主机（允许列表中的域名）。任何不在允许列表中的域名都应使用默认浏览器打开。

#### 请勿加载不可信的内容

如果可能，仅将严格限定范围的 URL 和应用程序开发者拥有的内容加载到 WebView 中。

#### 请勿泄露敏感数据

如果您的应用程序使用 WebView 访问敏感数据，请考虑在使用 JavaScript 接口之前，使用[`clearCache`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#clearCache\(boolean\)) 方法删除本地存储的任何文件。您还可以使用服务器端标头（例如 no-store）来指示应用程序不应缓存特定内容。

#### 请勿暴露敏感功能

如果您的应用程序需要敏感权限或收集敏感数据，请确保从应用程序内部的代码调用该应用程序，并向用户提供醒目的披露信息。避免使用 JavaScript 接口进行任何敏感操作或处理用户数据。

#### 以 API 级别 21 或更高级别为目标平台

使用 addJavascriptInterface 方法的一种安全方法是，确保该方法仅在 API 级别 21 或更高时调用，从而将目标 API 级别设为 21 或更高。在 API 21 之前，JavaScript 可以使用反射来访问注入对象的公共字段。

---

## 风险：MessageChannel 风险

`postWebMessage()` 和 `postMessage()` 中缺乏来源控制，可能会允许攻击者拦截消息或向本地处理程序发送消息。

### 缓解措施

设置 `postWebMessage()` 或 `postMessage()` 时，应避免使用 * 作为目标来源，而是显式指定预期的发送域，从而仅允许来自受信任域的消息。

---

## 资源

- [postMessage() 最佳实践](https://fastercapital.com/content/Channel-messaging--Best-Practices-for-Implementing-Channel-Messaging-in-JavaScript.html)
- [addJavascriptInterface 文档](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#addJavascriptInterface\(java.lang.Object,%20java.lang.String\))
- [postMessage() 文档](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage#the_dispatched_event)
- [WebMessagePort.postMessage() 文档](https://developer.android.com/reference/android/webkit/WebMessagePort?hl=zh-cn#postMessage\(android.webkit.WebMessage\))
- [WebViewClient.shouldInterceptRequest 文档](https://developer.android.com/reference/android/webkit/WebViewClient?hl=zh-cn#shouldInterceptRequest\(android.webkit.WebView,%20android.webkit.WebResourceRequest\))
- [有关 addJavascriptInterface 的安全建议文档](https://developer.android.com/docs/quality-guidelines/core-app-quality?hl=zh-cn#sc)
- [clearCache 文档](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#clearCache\(boolean\))
- [removeJavascript 文档](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn#removeJavascriptInterface\(java.lang.String\))
- [在 WebView 中启用 JavaScript](https://developer.android.com/develop/ui/views/layout/webapps/webview?hl=zh-cn#EnablingJavaScript)
