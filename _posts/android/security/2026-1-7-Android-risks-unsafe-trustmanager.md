---
title: Android-安全性-安全风险-不安全的 X.509 TrustManager
date: 2026-1-7 10:03:48 +0800
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
description: 不安全的 X509TrustManager 实现可能会导致漏洞，攻击者可以利用这些漏洞对来自受害应用的网络流量实施 MitM（中间人）攻击。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

`X509TrustManager` 类负责验证远程服务器的真实性。具体方法是验证服务器的证书。

Android 应用中的不安全 `X509TrustManager` 实现是指未能正确验证与应用程序通信的服务器的真实性的实现。这会使得攻击者能够冒充合法服务器，诱使应用向攻击者发送敏感数据。

由于使用 [`X509TrustManager`](https://developer.android.com/reference/javax/net/ssl/X509TrustManager?hl=zh-cn#checkServerTrusted\(java.security.cert.X509Certificate%5B%5D,%20java.lang.String\)) 类时，Java 和 Android 允许完全重写服务器验证机制。因此存在漏洞。`X509TrustManager` 类有有两个值得关注的函数：[`checkServerTrusted()`](https://developer.android.com/reference/javax/net/ssl/X509TrustManager?hl=zh-cn#checkServerTrusted\(java.security.cert.X509Certificate%5B%5D,%20java.lang.String\)) 和 [`getAcceptedIssuers()`](https://developer.android.com/reference/javax/net/ssl/X509TrustManager?hl=zh-cn#getAcceptedIssuers\(\))。这些函数调用可以配置为信任所有 X.509 证书。自定义验证逻辑可能存在缺陷或不完整，从而允许建立意外连接。在所有这些情况下，该类的目的都落空了，基于 `X509TrustManager` 输出建立的网络连接是不安全的。

## 影响

不安全的 `X509TrustManager` 实现可能会导致漏洞，攻击者可以利用这些漏洞对来自受害应用的网络流量实施 MitM（中间人）攻击。利用此不安全代码的影响是，如果触发此代码，用户的应用网络数据可能会因为遭受（远程或本地）网络攻击而泄露。具体影响取决于无意中被泄露的网络流量的内容（个人身份信息、私密信息、敏感会话值、服务凭据等）。

## 缓解措施

使用 [NetworkSecurityConfig.xml](https://developer.android.com/training/articles/security-config?hl=zh-cn) 功能来确保正确处理所有生产、测试、调试和开发阶段连接，而不是使用或实现自定义 TLS/SSL 证书验证代码。如果测试和调试 build 需要使用自签名证书，请考虑使用 NetworkSecurityConfig，而不是实现自定义 `X509TrustManager`。

## 资源

- [Play 警告文档](https://support.google.com/faqs/answer/6346016?hl=zh-cn)
- [用于协助配置网络安全配置 XML 文件的文档。](https://developer.android.com/training/articles/security-config?hl=zh-cn)
- [TrustManager 类的开发者文档。](https://developer.android.com/reference/javax/net/ssl/TrustManager?hl=zh-cn)
- [This check looks for X.509TrustManager implementations whose checkServerTrusted or checkClientTrusted methods do nothing (thus trusting any certificate chain).](https://googlesamples.github.io/android-custom-lint-rules/checks/TrustAllX509TrustManager.md.html)
- [This check looks for custom X.509TrustManager implementations.](https://googlesamples.github.io/android-custom-lint-rules/checks/CustomX509TrustManager.md.html)
- [https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/X509TrustManagerDetector.java](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/X509TrustManagerDetector.java)

