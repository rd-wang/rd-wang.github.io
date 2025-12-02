---
title: Android-安全风险-不安全的DNS设置
date: 2025-12-2 09:11:07 +0800
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
description: 如果恶意网络攻击者能够欺骗 DNS，他们就可以在不引起用户怀疑的情况下，悄悄将用户重定向到他们控制的网站。
math: true
---
# 不安全的 DNS 设置

**OWASP 类别：**[MASVS-NETWORK：网络通信](https://mas.owasp.org/MASVS/08-MASVS-NETWORK)

## 概览

当开发者自定义应用程序的 DNS 传输行为、绕过设备默认设置，或者用户在 Android 9 及更高版本中指定私有 DNS 服务器时，可能会出现不安全的 DNS 配置。偏离已知的良好 DNS 配置可能会使用户容易受到 DNS 欺骗或 DNS 缓存投毒等攻击，攻击者可以利用这些攻击将用户流量重定向到恶意网站。

## 影响

如果恶意网络攻击者能够伪造 DNS，他们就可以在不引起用户怀疑的情况下，悄悄地将用户重定向到他们控制的网站。例如，这个恶意网站可能会钓鱼用户以获取个人身份信息，导致用户无法访问服务，或者在不通知用户的情况下将用户重定向到其他网站。

## 风险：DNS 传输安全性存在漏洞

在 Android 9 及更高版本中，自定义 DNS 配置可能允许应用程序绕过 Android 内置的 DNS 传输安全机制。

### 缓解措施

#### 使用 Android OS 处理 DNS 流量

允许 Android 操作系统处理 DNS。自 SDK 28 版本起，Android 通过 DNS over TLS 为 DNS 传输增加了安全性，并在 SDK 30 版本中通过 DNS over HTTP/3 增加了安全性。

#### 使用 SDK 级别 >=28

将 SDK 级别更新为至少 28。请注意，此缓解措施需要与知名且安全的公共 DNS 服务器（例如[此处](https://dnsprivacy.org/public_resolvers/)所示的服务器）进行通信。

## 资源

- [解析 DNS 查询](https://developer.android.com/training/basics/network-ops/connecting?hl=zh-cn#lookup-dns)
- [DnsResolver 类的 Java 参考](https://developer.android.com/reference/android/net/DnsResolver?hl=zh-cn)
- [有关 DNS-over-HTTP/3 的 Android 安全博客文章](https://security.googleblog.com/2022/07/dns-over-http3-in-android.html)
- [DNS 安全传输概览](https://developers.google.com/speed/public-dns/docs/secure-transports?hl=zh-cn)
- [有关 DNS over TLS 的 Android 开发者博文](https://android-developers.googleblog.com/2018/04/dns-over-tls-support-in-android-p.html)

