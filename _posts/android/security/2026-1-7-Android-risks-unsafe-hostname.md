---
title: Android-安全风险-不安全的 HostnameVerifier
date: 2026-1-7 09:57:10 +0800
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
description: 不安全的 `HostnameVerifier` 实现可能会导致漏洞，攻击者可以利用这些漏洞对来自受害应用的网络流量实施 MiTM（中间人）攻击。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

[`HostnameVerifier`](https://developer.android.com/reference/javax/net/ssl/HostnameVerifier?hl=zh-cn#verify\(java.lang.String,%20javax.net.ssl.SSLSession\)) 实现负责验证服务器证书中的主机名是否与客户端尝试连接的服务器的主机名相匹配。

Android 应用中的不安全 `HostnameVerifier` 实现是指未能正确验证与应用程序通信的服务器的主机名的实现。这会使得攻击者能够冒充合法服务器，诱使应用向攻击者发送敏感数据。

之所以存在此漏洞，是因为 `HostnameVerifier` 类的函数调用可以跳过 X.509 证书的主机名验证，只验证该证书的哈希值。一个常见的误解是，[`SSLSession#isValid`](https://developer.android.com/reference/javax/net/ssl/SSLSession?hl=zh-cn#isValid\(\)) 函数会执行一项与安全相关的操作，但实际上，它的目的只是检查会话是否有效且可恢复或加入；而这两者都不能验证会话的安全性。`HostnameVerifier` 类已被 [NetworkSecurityConfig](https://developer.android.com/training/articles/security-config?hl=zh-cn) 所取代。

## 影响

不安全的 `HostnameVerifier` 实现可能会导致漏洞，攻击者可以利用这些漏洞对来自受害应用的网络流量实施 MiTM（中间人）攻击。利用这段不安全代码的后果是，如果这段代码被触发，网络攻击者（远程或本地）可能会窃取用户的应用程序网络数据。具体影响取决于无意中被泄露的网络流量的内容（个人身份信息、私密信息、敏感会话值、服务凭据等）。

## 缓解措施

使用 [NetworkSecurityConfig.xml](https://developer.android.com/training/articles/security-config?hl=zh-cn) 功能来确保正确处理所有生产、测试、调试和开发阶段连接，而不是使用或实现自定义 TLS/SSL 证书验证代码。

## 资源

- [网络安全配置文档](https://developer.android.com/training/articles/security-config?hl=zh-cn)
- [有关 `HostnameVerifier` 类的开发者文档](https://developer.android.com/reference/javax/net/ssl/HostnameVerifier?hl=zh-cn#verify\(java.lang.String,%20javax.net.ssl.SSLSession\))

