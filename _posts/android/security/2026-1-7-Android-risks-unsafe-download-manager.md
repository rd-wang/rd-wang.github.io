---
title: Android-安全风险-不安全的内容下载管理器
date: 2026-1-7 10:32:46 +0800
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
description: 使用 DownloadManager 可能会导致利用对外部存储空间的写入权限而导致漏洞。由于 android.permission.WRITE_EXTERNAL_STORAGE 权限允许对外部存储空间进行广泛访问，攻击者可能会静默修改文件和下载内容、安装可能存在恶意的应用、拒绝向核心应用提供服务，或导致应用崩溃。恶意攻击者还可以操纵发送到 Uri.parse() 的内容，以诱导用户下载有害文件。
math: true
---
**OWASP 类别：**[MASVS-NETWORK：网络通信](https://mas.owasp.org/MASVS/08-MASVS-NETWORK)

## 概览

DownloadManager 是在 API 级别 9 中引入的系统服务。它可处理长时间运行的 HTTP 下载，并允许应用作为后台任务下载文件。其 API 处理 HTTP 互动，在下载失败或连接发生更改以及系统重新启动后重新尝试下载。

DownloadManager 存在安全方面的缺陷，因此对于管理 Android 应用程序中的下载来说，它并非一个安全的选择。

**(1) 下载提供程序中的 CVE**

2018 年，我们在下载提供程序中发现并修复了三个 [CVE](https://ioactive.com/multiple-vulnerabilities-in-androids-download-provider-cve-2018-9468-cve-2018-9493-cve-2018-9546/)漏洞。下面简要介绍了每种方法（请参阅[技术详情](https://ioactive.com/multiple-vulnerabilities-in-androids-download-provider-cve-2018-9468-cve-2018-9493-cve-2018-9546/)）。

- **下载提供程序权限绕过** - 即使未获授权，恶意应用也可能会从下载提供程序检索所有条目，其中可能包括文件名、说明、标题、路径、网址等潜在敏感信息，以及对所有已下载文件的完整读写权限。恶意应用可能会在后台运行，监控所有下载内容并远程泄露其内容，或者在合法请求方访问文件之前动态修改文件。这可能会导致核心应用对用户进行拒绝服务攻击，包括无法下载更新。
- **下载提供程序 SQL 注入** - 通过 SQL 注入漏洞，无权限的恶意应用可以从下载提供程序检索所有条目。此外，具有有限权限的应用（例如 [`android.permission.INTERNET`](https://developer.android.com/reference/android/Manifest.permission#INTERNET)）也可以通过其他 URI 访问所有数据库内容。系统可能会检索潜在的敏感信息，例如文件名、说明、标题、路径、网址，并且根据权限，还可能会访问已下载的内容。
- **下载提供程序请求标头信息披露** - 获得 [`android.permission.INTERNET`](https://developer.android.com/reference/android/Manifest.permission#INTERNET) 权限的恶意应用可以检索下载提供程序请求标头表中所有条目。对于从 Android 浏览器、Google Chrome 或其他应用启动的任何下载，这些标头都可能包含敏感信息，例如会话 Cookie 或身份验证标头。这可能会让攻击者在从中获取敏感用户数据的任何平台上冒充用户。

**(2) 危险权限**

API 级别低于 29 的 DownloadManager 需要危险权限 - [`android.permission.WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission#WRITE_EXTERNAL_STORAGE)。对于 API 级别 29 及更高版本，无需 [`android.permission.WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission#WRITE_EXTERNAL_STORAGE) 权限，但 URI 必须引用应用拥有的目录中的路径，或顶级“下载”目录中的路径。

**(3) 依赖** `Uri.parse()`

DownloadManager 依赖于 `Uri.parse()` 方法来解析请求的下载内容的位置。为了提高性能，`Uri` 类对不可信输入的验证很少或根本不进行验证。

## 影响

使用 DownloadManager 可能会导致利用对外部存储空间的写入权限而导致漏洞。由于 android.permission.WRITE_EXTERNAL_STORAGE 权限允许对外部存储空间进行广泛访问，攻击者可能会静默修改文件和下载内容、安装可能存在恶意的应用、拒绝向核心应用提供服务，或导致应用崩溃。恶意攻击者还可以操纵发送到 Uri.parse() 的内容，以诱导用户下载有害文件。

## 缓解措施

请改用 HTTP 客户端（例如 Cronet）、进程调度程序/管理器，以及在网络连接中断时确保重试的方法，直接在应用中设置下载，而不是使用 DownloadManager。[该库的文档](https://developer.android.com/develop/connectivity/cronet?hl=zh-cn)包含指向[示例](https://github.com/GoogleChromeLabs/cronet-sample)应用的链接，以及有关如何实现该应用的[说明](https://developer.android.com/develop/connectivity/cronet/start?hl=zh-cn)。

如果您的应用需要能够管理进程调度、在后台运行下载，或在网络丢失后重新尝试建立下载，请考虑添加 [`WorkManager`](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn) 和 [`ForegroundServices`](https://developer.android.com/develop/background-work/services/foreground-services?hl=zh-cn)。

使用 Cronet 设置下载的示例代码如下，取自 Cronet [Codelab](https://developer.android.com/codelabs/cronet?hl=zh-cn#8)。

[Kotlin](https://developer.android.com/privacy-and-security/risks/unsafe-download-manager?hl=zh-cn#kotlin)
```kotlin
override suspend fun downloadImage(url: String): ImageDownloaderResult {
   val startNanoTime = System.nanoTime()
   return suspendCoroutine {
       cont ->
       val request = engine.newUrlRequestBuilder(url, object: ReadToMemoryCronetCallback() {
       override fun onSucceeded(
           request: UrlRequest,
           info: UrlResponseInfo,
           bodyBytes: ByteArray) {
           cont.resume(ImageDownloaderResult(
               successful = true,
               blob = bodyBytes,
               latency = Duration.ofNanos(System.nanoTime() - startNanoTime),
               wasCached = info.wasCached(),
               downloaderRef = this@CronetImageDownloader))
       }
       override fun onFailed(
           request: UrlRequest,
           info: UrlResponseInfo,
           error: CronetException
       ) {
           Log.w(LOGGER_TAG, "Cronet download failed!", error)
           cont.resume(ImageDownloaderResult(
               successful = false,
               blob = ByteArray(0),
               latency = Duration.ZERO,
               wasCached = info.wasCached(),
               downloaderRef = this@CronetImageDownloader))
       }
   }, executor)
       request.build().start()
   }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/unsafe-download-manager?hl=zh-cn#java)

```java
@Override
public CompletableFuture<ImageDownloaderResult> downloadImage(String url) {
    long startNanoTime = System.nanoTime();
    return CompletableFuture.supplyAsync(() -> {
        UrlRequest.Builder requestBuilder = engine.newUrlRequestBuilder(url, new ReadToMemoryCronetCallback() {
            @Override
            public void onSucceeded(UrlRequest request, UrlResponseInfo info, byte[] bodyBytes) {
                return ImageDownloaderResult.builder()
                        .successful(true)
                        .blob(bodyBytes)
                        .latency(Duration.ofNanos(System.nanoTime() - startNanoTime))
                        .wasCached(info.wasCached())
                        .downloaderRef(CronetImageDownloader.this)
                        .build();
            }
            @Override
            public void onFailed(UrlRequest request, UrlResponseInfo info, CronetException error) {
                Log.w(LOGGER_TAG, "Cronet download failed!", error);
                return ImageDownloaderResult.builder()
                        .successful(false)
                        .blob(new byte[0])
                        .latency(Duration.ZERO)
                        .wasCached(info.wasCached())
                        .downloaderRef(CronetImageDownloader.this)
                        .build();
            }
        }, executor);
        UrlRequest urlRequest = requestBuilder.build();
        urlRequest.start();
        return urlRequest.getResult();
    });
}
```

## 资源

- [DownloadManager 的主文档页面](https://developer.android.com/reference/android/app/DownloadManager?hl=zh-cn)
- [DownloadManager CVE 报告](https://ioactive.com/multiple-vulnerabilities-in-androids-download-provider-cve-2018-9468-cve-2018-9493-cve-2018-9546/)
- [Android 权限绕过漏洞 CVE 2018-9468](https://ioactive.com/wp-content/uploads/2019/04/IOActive-Security-Advisory-Androids-Download-Provider-Permission-Bypass-CVE-2018-9468.pdf)
- [Android 下载提供程序 SQL 注入 CVE-2018- 9493](https://act-on.ioactive.com/acton/attachment/34793/f-722b41b4-7aff-4b35-9925-c221a217744d/1/-/-/-/-/cve-2018-9493.pdf)
- [Android 下载提供程序权限绕过 CVE2018-9468](https://act-on.ioactive.com/acton/attachment/34793/f-3b8bb46b-d105-4efd-97a1-9970bfa6928b/1/-/-/-/-/cve-2018-9546.pdf)
- [Cronet 的主要文档页面](https://developer.android.com/develop/connectivity/cronet?hl=zh-cn)
- [在应用中使用 Cronet 的说明](https://developer.android.com/develop/connectivity/cronet/start?hl=zh-cn#java)
- [Cronet 实现示例](https://github.com/GoogleChromeLabs/cronet-sample)
- [URI 文档](https://developer.android.com/reference/android/net/Uri?hl=zh-cn)
- [ForegroundService 文档](https://developer.android.com/develop/background-work/services/foreground-services?hl=zh-cn)
- [WorkManager 文档](https://developer.android.com/reference/androidx/work/WorkManager?hl=zh-cn)

