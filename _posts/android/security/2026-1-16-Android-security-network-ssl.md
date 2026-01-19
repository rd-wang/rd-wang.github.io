---
title: Android-安全性-通过网络协议确保安全
date: 2026-1-16 14:19:24 +0800
categories:
  - Android
  - Security
  - SSL
tags:
  - Android
  - Security
  - SSL
  - Network
description: "客户端和服务器之间的加密互动使用传输层安全协议 (TLS) 来保护应用数据。\r\r本文介绍了与安全网络协议相关的最佳实践和公钥基础架构 (PKI) 注意事项。"
math: true
---
客户端和服务器之间的加密互动使用[传输层安全协议 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) 来保护应用数据。

本文介绍了与安全网络协议相关的最佳实践和[公钥基础架构 (PKI)](https://en.wikipedia.org/wiki/Public-key_infrastructure) 注意事项。如需了解详情，请阅读 [Android 安全性概览](https://source.android.com/tech/security/index.html?hl=zh-cn)以及[权限概览](https://developer.android.com/guide/topics/security/permissions?hl=zh-cn)。

## 概念

具有 TLS 证书的服务器拥有公钥和匹配的私钥。服务器在 TLS 握手期间使用[公钥加密](https://en.wikipedia.org/wiki/Public-key_cryptography)对其证书签名。

简单的握手只能证明服务器知道证书的私钥。为了解决此问题，请让客户端信任多个证书。如果指定服务器的证书未出现在客户端可信证书集中，则该服务器不可信。

但是，服务器可能会使用密钥轮替将证书的公钥更换为新的公钥。当服务器配置发生更改后，就需要更新客户端应用。如果服务器属于第三方网络服务（例如网络浏览器或电子邮件应用），则更难确定何时更新客户端应用。

服务器通常通过[证书授权机构 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 来颁发证书，这将确保客户端配置随着时间推移而更加稳定。CA 使用其私钥为服务器证书[签名](https://en.wikipedia.org/wiki/Digital_signature)。然后，客户端可以检查服务器是否具有平台已知的 CA 证书。

可信 CA 通常列在主机平台上。Android 8.0（API 级别 26）包含 100 多个 CA，这些 CA 在每个版本中都会更新，并且在不同设备之间保持一致。

客户端应用需要一种机制来验证服务器，因为 CA 为许多服务器提供证书。CA 证书使用特定名称（如 gmail.com）或使用通配符（如 *.google.com）来标识服务器。

如需查看网站的服务器证书信息，请使用 [`openssl`](http://www.openssl.org/docs/apps/openssl.html) 工具的 `s_client` 命令，传入端口号。默认情况下，HTTPS 使用端口 443。

该命令将 `openssl s_client` 的输出传输到 `openssl x509`，后者将根据 [X.509 标准](https://en.wikipedia.org/wiki/X.509)设置证书相关信息的格式。该命令会请求获取主题（服务器名称）和颁发者 (CA) 的信息。

```bash
openssl s_client -connect WEBSITE-URL:443 | \
  openssl x509 -noout -subject -issuer
```
## HTTPS 示例

假设您有一个由知名 CA 颁发证书的网络服务器，那么，您可以使用如下代码发起安全的请求：

[Kotlin](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn#kotlin)
```kotlin
val url = URL("https://wikipedia.org")
val urlConnection: URLConnection = url.openConnection()
val inputStream: InputStream = urlConnection.getInputStream()
copyInputStreamToOutputStream(inputStream, System.out)
```
[Java](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn#java)

```java
URL url = new URL("https://wikipedia.org");
URLConnection urlConnection = url.openConnection();
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);
```

如需自定义 HTTP 请求，请转换为 [`HttpURLConnection`](https://developer.android.com/reference/java/net/HttpURLConnection?hl=zh-cn)。Android `HttpURLConnection` 文档提供了有关处理请求和响应标头、发布内容、管理 Cookie、使用代理、缓存响应等的示例。Android 框架使用这些 API 验证证书和主机名。

请尽可能使用这些 API。以下部分介绍了一些常见问题，需要采用不同的解决方案。

## 验证服务器证书时的常见问题

假设 [`getInputStream()`](https://developer.android.com/reference/java/net/URLConnection?hl=zh-cn#getInputStream\(\)) 没有返回内容，而是抛出了异常：

```java
javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
        at org.apache.harmony.xnet.provider.jsse.OpenSSLSocketImpl.startHandshake(OpenSSLSocketImpl.java:374)
        at libcore.net.http.HttpConnection.setupSecureSocket(HttpConnection.java:209)
        at libcore.net.http.HttpsURLConnectionImpl$HttpsEngine.makeSslConnection(HttpsURLConnectionImpl.java:478)
        at libcore.net.http.HttpsURLConnectionImpl$HttpsEngine.connect(HttpsURLConnectionImpl.java:433)
        at libcore.net.http.HttpEngine.sendSocketRequest(HttpEngine.java:290)
        at libcore.net.http.HttpEngine.sendRequest(HttpEngine.java:240)
        at libcore.net.http.HttpURLConnectionImpl.getResponse(HttpURLConnectionImpl.java:282)
        at libcore.net.http.HttpURLConnectionImpl.getInputStream(HttpURLConnectionImpl.java:177)
        at libcore.net.http.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:271)
```

出现这种情况的原因有很多，其中包括：

1. [颁发服务器证书的 CA 未知](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn#UnknownCa)。
2. [服务器证书不是 CA 签名的，而是自签名的](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn#SelfSigned)。
3. [服务器配置缺少中间 CA](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn#MissingCa)。

下面几部分将讨论如何解决这些问题，同时确保与服务器的连接安全无虞。

### 未知的证书授权机构

出现 [`SSLHandshakeException`](https://developer.android.com/reference/javax/net/ssl/SSLHandshakeException?hl=zh-cn) 是因为系统不信任 CA。原因可能是您有一个由 Android 尚不信任的新 CA 颁发的证书，或您的应用在没有 CA 的较旧版本上运行。由于 CA 的私有性质，人们对其知之甚少。CA 未知的原因通常是它不是公共 CA，而是由政府、公司或教育机构等组织颁发的仅供其自己使用的私有 CA。

如需信任自定义 CA，而无需更改应用代码，请更改您的[网络安全配置](https://developer.android.com/training/articles/security-config?hl=zh-cn#TrustingAdditionalCas)。

>**注意：** 很多网站都会介绍一种效果不佳的替代解决方案，即让您安装一个不起作用的 [`TrustManager`](https://developer.android.com/reference/javax/net/ssl/TrustManager?hl=zh-cn) 。这样做会使用户在使用公共 Wi-Fi 热点时容易受到攻击，因为攻击者可以使用 DNS 技巧将用户的流量发送到伪装成您的服务器的代理。攻击者随后可以记录密码和其他个人数据。这样做之所以有效，是因为攻击者可以生成证书，而如果没有 TrustManager 来验证证书是否来自可信来源，就无法阻止此类攻击。所以，即使是暂时的，也不要这样做。相反，应该让你的应用信任服务器证书的颁发者。

### 自签名的服务器证书

其次，由于使用了自签名证书，服务器成为了自己的 CA，因此可能会出现 [`SSLHandshakeException`](https://developer.android.com/reference/javax/net/ssl/SSLHandshakeException?hl=zh-cn)。这类似于未知的证书颁发机构，因此请修改应用程序的网络安全配置，以信任您的自签名证书。

### 缺少中间证书授权机构

第三，由于缺少中间 CA，会出现 [`SSLHandshakeException`](https://developer.android.com/reference/javax/net/ssl/SSLHandshakeException?hl=zh-cn)。公共 CA 很少签署服务器证书，通常情况下，根 CA 会签署中间 CA 证书。

为了降低被攻击的风险，证书颁发机构（CA）会将根CA保持离线状态。然而，像Android这样的操作系统通常只直接信任根CA，在服务器证书（由中间 CA 签名）和证书验证器（识别根 CA）之间留下一个短暂的信任缺口。

为了消除此信任缺口，服务器在 TLS 握手期间，会将一串证书从服务器 CA 经由任何中间证书发送到受信任的根 CA。

例如，下面是通过 [`openssl`](http://www.openssl.org/docs/apps/openssl.html) `s_client` 命令看到的 mail.google.com 证书链：

```bash
$ openssl s_client -connect mail.google.com:443
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google LLC/CN=mail.google.com
   i:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
 1 s:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
   i:/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
---
```

这表明服务器会发送由 Thawte SGC CA（中间 CA）颁发的 mail.google.com 证书，以及由 Verisign CA（Android 信任的主 CA）颁发的 Thawte SGC CA 的第二个证书。

但是，服务器可能未配置必要的中间 CA。例如，以下服务器可能会导致 Android 浏览器出现错误，并在 Android 应用中引发异常：

```bash
$ openssl s_client -connect egov.uscis.gov:443
---
Certificate chain
 0 s:/C=US/ST=District Of Columbia/L=Washington/O=U.S. Department of Homeland Security/OU=United States Citizenship and Immigration Services/OU=Terms of use at www.verisign.com/rpa (c)05/CN=egov.uscis.gov
   i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)10/CN=VeriSign Class 3 International Server CA - G3
---
```

与未知的 CA 或自签名服务器证书不同，大多数桌面浏览器在与此服务器通信时不会出现错误。桌面浏览器会缓存可信的中间 CA。浏览器从一个网站了解到中间 CA 之后，证书链中就不再需要它了。

有些网站故意使用辅助服务器来提供资源。为了节省带宽，它们可能会使用具有完整证书链的服务器来提供主页的 HTML 页面，但图片、CSS 和 JavaScript 则不使用证书颁发机构 (CA) 的证书。遗憾的是，有时这些服务器可能正在提供您尝试从 Android 应用访问的 Web 服务，而 Android 应用的容错性并不高。

要解决此问题，请将服务器配置为在服务器链中包含中间 CA。大多数 CA 都提供了针对常见 Web 服务器的配置说明。

## 有关直接使用 SSLSocket 的警告

到目前为止，所举示例都侧重于使用 [`HttpsURLConnection`](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn) 的 HTTPS。有时，应用需要单独使用 TLS 与 HTTPS。例如，某个电子邮件应用可能使用 TLS 的变体 SMTP、POP3 或 IMAP。在这些情况下，应用可以直接使用 [`SSLSocket`](https://developer.android.com/reference/javax/net/ssl/SSLSocket?hl=zh-cn)，与 `HttpsURLConnection` 在内部执行的操作非常相似。

目前为止所介绍的用于处理证书验证问题的技术也适用于 `SSLSocket`。事实上，使用自定义 [`TrustManager`](https://developer.android.com/reference/javax/net/ssl/TrustManager?hl=zh-cn) 时，传递到 `HttpsURLConnection` 的是 [`SSLSocketFactory`](https://developer.android.com/reference/javax/net/ssl/SSLSocketFactory?hl=zh-cn)。因此，如果需要结合使用自定义 `TrustManager` 和 `SSLSocket`，请遵循相同的步骤，并使用 `SSLSocketFactory` 创建您的 `SSLSocket`。

> **注意：** `SSLSocket` **不会**执行主机名验证。您的应用程序需要自行进行主机名验证，最好是通过调用[`getDefaultHostnameVerifier()`](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn#getDefaultHostnameVerifier\(\)) 并传入预期的主机名来实现。另外，请注意，[`HostnameVerifier.verify()`](https://developer.android.com/reference/javax/net/ssl/HostnameVerifier?hl=zh-cn#verify\(java.lang.String,%20javax.net.SSL.SSLSession\)) 函数在出错时不会抛出异常，而是返回一个布尔值结果，您必须显式地检查该结果。

## 证书验证

TLS 依赖于证书颁发机构 (CA) 仅向经过验证的服务器和域名所有者颁发证书。在极少数情况下，CA 会被欺骗，或者像 [Comodo](https://en.wikipedia.org/wiki/Comodo_Group#Breach_of_security) 或 [DigiNotar](https://en.wikipedia.org/wiki/DigiNotar) 那样遭到入侵。导致主机名的证书颁发给服务器或域名的所有者以外的其他人。

为了降低这种风险，Android 通过黑名单和证书透明性的结合，在系统范围内处理证书吊销，而无需依赖在线证书验证。此外，Android 还会验证附加到 TLS 握手中的 OCSP 响应。

要在您的应用中启用证书透明度，请参阅我们的网络安全配置文档中的[选择加入证书透明度](https://developer.android.com/privacy-and-security/security-config#CertificateTransparencySummary)部分。
## 限制应用仅使用特定证书

> **注意：** 证书锁定（即限制应用程序只能使用之前授权的证书）这种做法不建议用于 Android 应用程序。未来的服务器配置更改，例如更改为另一个 CA，会导致具有固定证书的应用程序在未收到客户端软件更新的情况下无法连接到服务器。

如需将应用限制为仅接受您指定的证书，请务必添加多个备用 PIN 码（其中至少包括一个完全由您控制的密钥），并设置足够短的有效期以防止兼容性问题。[网络安全配置](https://developer.android.com/training/articles/security-config?hl=zh-cn#CertificatePinning)中提供了这些固定功能。

## 客户端证书

本文重点介绍了如何使用 TLS 来确保与服务器之间的通信安全。TLS 也支持客户端证书的概念，允许服务器验证客户端的身份。虽然这超出了本文的范围，但其中涉及的技术与指定自定义 `TrustManager` 类似。

## Nogotofail：网络流量安全测试工具 

Nogotofail 是一款工具，可让您轻松确认您的应用程序是否安全，免受已知的 TLS/SSL 漏洞和错误配置的影响。它是一款自动化、功能强大且可扩展的工具，可用于测试任何可通过其网络流量进行测试的设备的网络安全问题。

Nogotofail 可用于三个主要用例：

- 查找 bug 和漏洞。
- 验证修复并监测回归。
- 了解哪些应用和设备正在生成哪些流量。

Nogotofail 适用于 Android、iOS、Linux、Windows、ChromeOS 和 macOS 操作系统。事实上，任何用于连接互联网的设备都可以使用 Nogotofail。客户端可用于在 Android 和 Linux 上配置设置和接收通知，攻击引擎本身可以部署为路由器、VPN 服务器或代理。

您可以在 [Nogotofail 开源项目](https://github.com/google/nogotofail)网站上访问此工具。

## SSL 和 TLS 更新

### Android 10

当 TLS 服务器在 TLS 握手中发送证书请求消息时，某些浏览器（如 Google Chrome）允许用户选择证书。从 Android 10 开始，KeyChain 对象在调用 `KeyChain.choosePrivateKeyAlias()` 向用户显示证书选择提示时，会遵循颁发者和密钥规范参数。尤其需要注意的是，此提示不包含任何不符合服务器规范的选项。

如果没有用户可选择的证书可用，例如没有证书与服务器规范匹配，或者设备没有安装任何证书，则根本不会出现证书选择提示。

此外，在 Android 10 或更高版本中，无需锁定设备屏幕即可将密钥或 CA 证书导入 KeyChain 对象。

#### TLS 1.3 默认处于启用状态

在 Android 10 及更高版本中，所有 TLS 连接默认启用 TLS 1.3。以下是关于我们 TLS 1.3 实现的一些重要细节：

- TLS 1.3 加密套件不可自定义。在启用 TLS 1.3 后，受支持的 TLS 1.3 加密套件会始终保持启用状态。任何尝试通过调用 [`setEnabledCipherSuites()`](https://developer.android.com/reference/javax/net/ssl/SSLSocket?hl=zh-cn#setEnabledCipherSuites\(java.lang.String%5B%5D\)) 停用该加密套件的操作均会被忽略。
- 在协商 TLS 1.3 时，系统会在将会话添加到会话缓存之前调用 [`HandshakeCompletedListener`](https://developer.android.com/reference/javax/net/ssl/HandshakeCompletedListener?hl=zh-cn) 对象。（在 TLS 1.2 和之前的其他版本中，这些对象是在将会话添加到会话缓存之后调用的。）
- 在某些情况下，SSLEngine 实例会在之前的 Android 版本中抛出 [`SSLHandshakeException`](https://developer.android.com/reference/javax/net/ssl/SSLHandshakeException?hl=zh-cn)，而这些实例在 Android 10 及更高版本中会改为抛出 [`SSLProtocolException`](https://developer.android.com/reference/javax/net/ssl/SSLProtocolException?hl=zh-cn)。
- 不支持 0-RTT 模式。

如有需要，您可以通过调用 [`SSLContext.getInstance("TLSv1.2")`](https://developer.android.com/reference/javax/net/ssl/SSLContext?hl=zh-cn#getInstance\(java.lang.String\)) 来获取已停用 TLS 1.3 的 SSLContext。您还可以对相关对象调用 [`setEnabledProtocols()`](https://developer.android.com/reference/javax/net/ssl/SSLSocket?hl=zh-cn#setEnabledProtocols\(java.lang.String%5B%5D\))，从而为每个连接启用或停用协议版本。

#### TLS 不信任使用 SHA-1 签名的证书

在 Android 10 中，使用 SHA-1 哈希算法的证书在 TLS 连接中不受信任。自 2016 年以来，根 CA 未再颁发过此类证书，因为它们不再受 Chrome 或其他主流浏览器的信任。

如果某网站使用的是 SHA-1 证书，则任何尝试连接该网站的操作都将失败。

#### KeyChain 行为变更和改进

当 TLS 服务器在 TLS 握手中发送证书请求消息时，某些浏览器（如 Google Chrome）允许用户选择证书。从 Android 10 开始，[`KeyChain`](https://developer.android.com/reference/android/security/KeyChain?hl=zh-cn) 对象在调用 `KeyChain.choosePrivateKeyAlias()` 向用户显示证书选择提示时，会遵循颁发者和密钥规范参数。尤其需要注意的是，此提示不包含任何不符合服务器规范的选项。

如果没有用户可选择的证书可用，例如没有证书与服务器规范匹配，或者设备没有安装任何证书，则根本不会出现证书选择提示。

此外，在 Android 10 或更高版本中，无需锁定设备屏幕即可将密钥或 CA 证书导入 KeyChain 对象。

#### 其他 TLS 和加密更改

Android 10 中引入的 TLS 和加密库方面的一些细小变更包括：

- AES/GCM/NoPadding 和 ChaCha20/Poly1305/NoPadding 加密会从 `getOutputSize()` 中返回更准确的缓冲区大小。
- 使用 TLS 1.2 或更高版本的最高协议在尝试连接时会忽略 TLS_FALLBACK_SCSV 加密套件。由于 TLS 服务器实现方面的改进，我们不建议尝试 TLS 外部回退。不过，我们建议依赖于 TLS 版本协商。
- ChaCha20-Poly1305 是 ChaCha20/Poly1305/NoPadding 的别名。
- 带有尾随点的主机名不属于有效的 SNI 主机名。
- 为证书响应选择签名密钥时，将遵循 CertificateRequest 中的 supported_signature_algorithms 扩展。
- 不透明的签名密钥（如 Android 密钥库中的密钥）可在 TLS 中与 RSA-PSS 签名一起使用。

#### HTTPS 连接变更

如果在 Android 10 上运行的应用将 null 传递给 [`setSSLSocketFactory()`](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn#setSSLSocketFactory\(javax.net.ssl.SSLSocketFactory\))，则会出现 [`IllegalArgumentException`](https://developer.android.com/reference/java/lang/IllegalArgumentException?hl=zh-cn)。在以前的版本中，将 null 传递给 `setSSLSocketFactory()` 与传入当前的[默认工厂](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn#getDefaultSSLSocketFactory\(\))效果相同。

### Android 11

#### SSL 套接字默认情况下使用 Conscrypt SSL 引擎

Android 的默认 SSLSocket 实现基于 [`Conscrypt`](https://github.com/google/conscrypt)。从 Android 11 开始，该实现是基于 Conscrypt 的 SSLEngine 在内部构建而成的。