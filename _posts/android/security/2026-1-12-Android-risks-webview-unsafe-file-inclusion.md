---
title: Android-安全风险-不安全的文件包含
date: 2026-1-12 10:25:17 +0800
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
description: 文件包含的影响可能取决于在 WebView 中配置的 WebSettings。过于宽泛的文件权限可能会让攻击者访问本地文件并窃取敏感数据、PII（个人身份信息）或私密应用数据。
math: true
---
**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

本文介绍了与文件包含相关的几个问题，这些问题具有类似的缓解措施。这些问题主要涉及因访问 WebView 中的文件而产生的漏洞，从允许文件访问或启用 JavaScript 的危险 [`WebSettings`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn) 到创建文件选择请求的 WebKit 方法，都可能存在漏洞。如果您想寻求有关修复 WebView 中因使用 `file://` 方案、不受限制地访问本地文件和跨站脚本攻击而引起的问题的指导，本应该会有所帮助。

具体而言，本文涵盖以下主题：

- `WebSettings` 是一个包含用于管理 WebView 设置状态的方法的类。这些方法可能会使 WebView 遭受不同的攻击，我们将在下文中对此进行概述。在本文中，我们将介绍与文件访问方式相关的方法，以及允许执行 JavaScript 的设置：
- [`setAllowFileAccess`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowFileAccess\(boolean\))、[`setAllowFileAccessFromFileURLs`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowFileAccessFromFileURLs\(boolean\)) 和 [`setAllowUniversalAccessFromFileURLs`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowUniversalAccessFromFileURLs\(boolean\)) 方法可用于使用文件方案网址 (`file://`) 授予对本地文件的访问权限。不过，恶意脚本可能会利用这些方法来访问应用有权访问的任意本地文件，例如其自己的 `/data/` 文件夹。因此，这些方法已被标记为不安全，并在 API 30 中被弃用，取而代之的是更安全的替代方法，例如 [`WebViewAssetLoader`](https://developer.android.com/reference/androidx/webkit/WebViewAssetLoader?hl=zh-cn)。
- [`setJavascriptEnabled`](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setJavaScriptEnabled\(boolean\)) 方法可用于在 WebView 中启用 JavaScript 执行。这使得应用容易受到文件级 XSS 攻击。尤其是当 WebView 配置为允许加载本地文件或可能包含可执行代码的不受信任的网络内容、允许访问可由外部来源创建或更改的文件，或者允许 WebView 执行 JavaScript 时，用户及其数据会面临风险。
- [`WebChromeClient.onShowFileChooser`](https://developer.android.com/reference/android/webkit/WebChromeClient?hl=zh-cn#onShowFileChooser\(android.webkit.WebView,%20android.webkit.ValueCallback%3Candroid.net.Uri%5B%5D%3E,%20android.webkit.WebChromeClient.FileChooserParams\)) 是 `android.webkit` 软件包中的一种方法，该软件包提供网页浏览工具。此方法可用于允许用户在 WebView 中选择文件。不过，由于 WebView 不会强制限制选择的文件，因此此功能可能会被滥用。

## 影响

文件包含的影响可能取决于在 WebView 中配置的 WebSettings。过于宽泛的文件权限可能会让攻击者访问本地文件并窃取敏感数据、PII（个人身份信息）或私密应用数据。启用 JavaScript 执行功能可能会导致攻击者在 WebView 内或用户设备上运行 JavaScript。使用 `onShowFileChooser` 方法选择的文件可能会损害用户安全性，因为该方法或 WebView 无法确保文件来源可信。

## 风险：通过 file:// 对文件进行风险访问

启用 `setAllowFileAccess`、`setAllowFileAccessFromFileURLs` 和 `setAllowUniversalAccessFromFileURLs` 可能会导致具有 `file://` 上下文的恶意 intent 和 WebView 请求访问任意本地文件，包括 WebView Cookie 和应用私有数据。此外，使用 `onShowFileChooser` 方法可让用户选择并下载来源不受信任的文件。

根据应用配置，这些方法都可能导致个人身份信息、登录凭据或其他敏感数据被窃取。

### 缓解措施

#### 验证文件网址

如果您的应用需要通过 `file://` 网址访问文件，请务必仅将已知为合法的特定网址列入许可名单，避免出现[常见错误](https://blog.oversecured.com/Android-security-checklist-webview/)。

#### 使用 WebViewAssetLoader

请使用 [`WebViewAssetLoader`](https://developer.android.com/reference/androidx/webkit/WebViewAssetLoader?hl=zh-cn)，而不是上述方法。此方法使用 `http(s)//:` 方案（而非 `file://` 方案）来访问本地文件系统资源，因此不会受到所述攻击的影响。

[Kotlin](https://developer.android.com/privacy-and-security/risks/webview-unsafe-file-inclusion?hl=zh-cn#kotlin)
```kotlin
val assetLoader: WebViewAssetLoader = Builder()
  .addPathHandler("/assets/", AssetsPathHandler(this))
  .build()

webView.setWebViewClient(object : WebViewClientCompat() {
  @RequiresApi(21)
  override fun shouldInterceptRequest(view: WebView?, request: WebResourceRequest): WebResourceResponse {
    return assetLoader.shouldInterceptRequest(request.url)
  }

  @Suppress("deprecation") // for API < 21
  override fun shouldInterceptRequest(view: WebView?, url: String?): WebResourceResponse {
    return assetLoader.shouldInterceptRequest(Uri.parse(url))
  }
})

val webViewSettings: WebSettings = webView.getSettings()
// Setting this off for security. Off by default for SDK versions >= 16.
webViewSettings.allowFileAccessFromFileURLs = false
// Off by default, deprecated for SDK versions >= 30.
webViewSettings.allowUniversalAccessFromFileURLs = false
// Keeping these off is less critical but still a good idea, especially if your app is not
// using file:// or content:// URLs.
webViewSettings.allowFileAccess = false
webViewSettings.allowContentAccess = false

// Assets are hosted under http(s)://appassets.androidplatform.net/assets/... .
// If the application's assets are in the "main/assets" folder this will read the file
// from "main/assets/www/index.html" and load it as if it were hosted on:
// https://appassets.androidplatform.net/assets/www/index.html
webView.loadUrl("https://appassets.androidplatform.net/assets/www/index.html")
```
[Java](https://developer.android.com/privacy-and-security/risks/webview-unsafe-file-inclusion?hl=zh-cn#java)

```java
final WebViewAssetLoader assetLoader = new WebViewAssetLoader.Builder()
         .addPathHandler("/assets/", new AssetsPathHandler(this))
         .build();

webView.setWebViewClient(new WebViewClientCompat() {
    @Override
    @RequiresApi(21)
    public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
        return assetLoader.shouldInterceptRequest(request.getUrl());
    }

    @Override
    @SuppressWarnings("deprecation") // for API < 21
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
        return assetLoader.shouldInterceptRequest(Uri.parse(url));
    }
});

WebSettings webViewSettings = webView.getSettings();
// Setting this off for security. Off by default for SDK versions >= 16.
webViewSettings.setAllowFileAccessFromFileURLs(false);
// Off by default, deprecated for SDK versions >= 30.
webViewSettings.setAllowUniversalAccessFromFileURLs(false);
// Keeping these off is less critical but still a good idea, especially if your app is not
// using file:// or content:// URLs.
webViewSettings.setAllowFileAccess(false);
webViewSettings.setAllowContentAccess(false);

// Assets are hosted under http(s)://appassets.androidplatform.net/assets/... .
// If the application's assets are in the "main/assets" folder this will read the file
// from "main/assets/www/index.html" and load it as if it were hosted on:
// https://appassets.androidplatform.net/assets/www/index.html
webview.loadUrl("https://appassets.androidplatform.net/assets/www/index.html");
```

#### 停用危险的 WebSettings 方法

方法 `setAllowFileAccess()`、`setAllowFileAccessFromFileURLs()` 和 `setAllowUniversalAccessFromFileURLs()` 的值在 API 级别 29 及更低级别中默认设置为 `TRUE`，在 API 级别 30 及更高级别中默认设置为 `FALSE`。

如果需要配置其他 `WebSettings`，最好**明确**停用这些方法，尤其是对于以不高于 29 的 API 级别为目标平台的应用。

---

## 风险：文件级 XSS

将 `setJavacriptEnabled` 方法设置为 `TRUE` 可在 WebView 中执行 JavaScript，如果同时启用文件访问权限（如前所述），则可以通过在任意文件中执行代码或在 WebView 中打开恶意网站来实施基于文件的 XSS 攻击。

### 缓解措施

#### 防止 WebView 加载本地文件

与之前的风险一样，如果将 `setAllowFileAccess()`、`setAllowFileAccessFromFileURLs()` 和 `setAllowUniversalAccessFromFileURLs()` 设置为 `FALSE`，则可以避免文件级 XSS。

#### 防止 WebView 执行 JavaScript

将方法 `setJavascriptEnabled` 设置为 `FALSE`，以便无法在 WebView 中执行 JavaScript。

#### 确保 WebView 不会加载不受信任的内容

有时，在 WebView 中启用这些设置是必要的。在这种情况下，请务必确保仅加载可信内容。限制 JavaScript 的执行范围，只允许执行你控制的 JavaScript 代码，并禁止执行任意 JavaScript 代码，是确保内容可信度的一个很好的方法。否则，阻止明文流量加载可以确保具有危险设置的 WebView 至少无法加载 HTTP URL。这可以通过清单来完成，即将 [`android:usesCleartextTraffic`](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#usesCleartextTraffic) 设置为 `False`，或者通过设置[`Network Security Config`](https://developer.android.com/training/articles/security-config?hl=zh-cn)来禁止 HTTP 流量。

---

## 资源

- [setAllowUniversalAccessFromFile网址s API 参考页面](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowUniversalAccessFromFileURLs\(boolean\))
- [setAllowFileAccessFromFile网址s API 参考页面](https://developer.android.com/reference/android/webkit/WebSettings?hl=zh-cn#setAllowFileAccessFromFileURLs\(boolean\))
- [WebViewAssetLoader API 参考页面](https://developer.android.com/reference/androidx/webkit/WebViewAssetLoader?hl=zh-cn)
- [CodeQL 文档](https://codeql.github.com/codeql-query-help/java/java-android-websettings-file-access/)
- [Oversecured 博客](https://blog.oversecured.com/Android-security-checklist-webview/#attacks-where-universal-file-access-from-file-urls-is-enabled)
- [onShowFileChooser 参考页面](https://developer.android.com/reference/android/webkit/WebChromeClient?hl=zh-cn#onShowFileChooser\(android.webkit.WebView,%20android.webkit.ValueCallback%3Candroid.net.Uri%5B%5D%3E,%20android.webkit.WebChromeClient.FileChooserParams\))

