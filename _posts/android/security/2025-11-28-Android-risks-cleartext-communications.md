---
title: Android-安全风险-明文通信
date: 2025-11-28 11:28:04 +0800
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
description: 当 Android 应用通过网络发送或接收明文数据时，监控网络的任何人都可以拦截并读取这些数据。如果这些数据包含敏感信息（例如密码、信用卡号或个人消息），可能会导致身份窃取、金融欺诈和其他严重问题。
math: true
---

# 明文通信

**OWASP 类别：**[MASVS-NETWORK：网络通信](https://mas.owasp.org/MASVS/08-MASVS-NETWORK)

## 概览

在 Android 应用中允许明文网络通信，意味着监控网络流量的任何人都可以查看和操纵正在传输的数据。如果传输的数据包含敏感信息（例如密码、信用卡号或其他个人信息），这就会成为一个漏洞。

无论您是否发送敏感信息，使用明文都可能是一个漏洞，因为明文流量也可能通过网络攻击（例如 ARP 或 DNS 投毒）来操纵，因此可能会使攻击者影响应用的行为。

## 影响

当 Android 应用通过网络发送或接收明文数据时，监控网络的任何人都可以拦截并读取这些数据。如果这些数据包含敏感信息（例如密码、信用卡号或个人消息），可能会导致身份窃取、金融欺诈和其他严重问题。

例如，应用如果以明文形式传输密码，可能会将这些凭据泄露给拦截流量的恶意操作者。然后，攻击者可能会利用这些数据在未经授权的情况下访问用户账号。

## 风险：未加密的通信渠道

通过未加密的通信通道传输数据会泄露设备和应用端点之间共享的数据。上述数据可能会被攻击者拦截并可能被修改。

### 缓解措施

数据应通过加密的通信通道发送。对于不提供加密功能的协议，应使用安全协议的替代方案。

## 特定风险

本部分将汇总符合以下条件的风险：需要采用非标准的缓解策略，或原本已在特定 SDK 级别得到缓解，而为了提供完整信息才列在此处。

### 风险信号：HTTP

此部分中的指南仅适用于以 Android 8.1（API 级别 27）或更低版本为目标平台的应用。从 Android 9（API 级别 28）开始，网址Connection、[Cronet](https://developer.android.com/develop/connectivity/cronet?hl=zh-cn) 和 [OkHttp](https://square.github.io/okhttp/) 等 HTTP 客户端会强制使用 HTTPS，因此系统默认情况下已停用明文支持。不过，请注意，其他 HTTP 客户端库（例如 [Ktor](https://ktor.io/)）不太可能对明文强制执行这些限制，因此应谨慎使用。

#### 缓解措施

使用 [NetworkSecurityConfig.xml](https://developer.android.com/training/articles/security-config?hl=zh-cn#CleartextTrafficPermitted) 功能来选择停用明文流量并为应用强制执行 HTTPS，仅为所需的特定网域保留例外情况（通常出于调试目的）：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">debug.domain.com</domain>
    </domain-config>
</network-security-config>
```

此选项有助于防止应用因外部源（如后端服务器）提供的网址发生变化而意外回归。

---

### 风险：FTP

使用 FTP 协议在设备之间交换文件会带来多种风险，其中最严重的是通信通道缺少加密。应改用更安全的替代方案，例如 SFTP 或 HTTPS。

#### 缓解措施

在应用中通过互联网实现数据交换机制时，您应使用 HTTPS 等安全协议。Android 提供了[一组 API](https://developer.android.com/develop/connectivity/network-ops/connecting?hl=zh-cn)，支持开发者创建客户端-服务器逻辑。您可以使用[传输层安全协议 (TLS)](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn) 来实现安全保护，确保两个端点之间的数据交换经过加密，从而防止恶意用户窃听通信内容和检索敏感数据。

通常，客户端-服务器架构依赖于开发者拥有的 API。如果您的应用依赖于一组 API 端点，请遵循以下安全最佳实践来保护 HTTPS 通信，从而确保深度安全：

- 身份验证 - 用户应使用 [OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/native-app?hl=zh-cn) 等安全机制进行身份验证。通常不建议使用基本身份验证，因为它不提供会话管理机制，并且如果凭据存储不当，可以从 Base64 解码。
- 授权 - 应根据最小权限原则限制用户仅访问预期资源。为应用的资源采用谨慎的访问权限控制解决方案即可实现这一点。
- 确保使用经过深思熟虑且最新的加密套件，遵循安全性最佳实践。例如，考虑支持 [TLSv1.3 协议](https://wiki.mozilla.org/Security/Server_Side_TLS)，并在必要时为 HTTPS 通信提供向后兼容性。

---

### 风险：自定义通信协议

实现自定义通信协议或尝试手动实现知名协议都很危险。

虽然自定义协议可让开发者根据预期需求定制独特的解决方案，但开发过程中发生的任何错误都可能导致安全漏洞。例如，在开发会话处理机制时出现错误，可能会导致攻击者能够窃听通信内容，并动态检索敏感信息。

另一方面，如果不使用操作系统或维护良好的第三方库来实现 HTTPS 等众所周知的协议，则会增加引入编码错误的可能性，这可能会导致在需要时难以（如果不是完全不可能的话）更新您实现的协议。此外，这可能会引入与使用自定义协议相同类型的安全漏洞。

#### 缓解措施

##### 使用维护的库实现众所周知的通信协议

如需在应用中实现 HTTPS 等众所周知的协议，应使用操作系统库或受维护的第三方库。

这样一来，开发者便可以安全地选择经过全面测试、随着时间的推移不断改进并且会持续接收安全更新以修复常见漏洞的解决方案。

此外，通过选择众所周知的协议，开发者可以获享跨各种系统、平台和 IDE 的广泛兼容性，从而降低开发过程中出现人为错误的可能性。

##### 使用 SFTP

此协议会对传输中的数据进行加密。使用此类文件交换协议时，应考虑采取其他措施：

- SFTP 支持不同类型的身份验证。应使用公钥身份验证方法，而不是基于密码的身份验证。此类密钥应以安全的方式创建和存储，建议为此使用 [Android 密钥库](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)。
- 确保支持的密码遵循安全最佳实践。

---

## 资源

- [Ktor](https://ktor.io/)
- [使用 Cronet 执行网络操作](https://developer.android.com/develop/connectivity/cronet?hl=zh-cn)
- [OkHttp](https://square.github.io/okhttp/)
- [为网络安全配置选择停用明文流量](https://developer.android.com/training/articles/security-config?hl=zh-cn#CleartextTrafficPermitted)
- [连接到网络](https://developer.android.com/develop/connectivity/network-ops/connecting?hl=zh-cn)
- [通过网络协议确保安全](https://developer.android.com/privacy-and-security/security-ssl?hl=zh-cn)
- [适用于移动应用和桌面应用的 OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/native-app?hl=zh-cn)
- [通过 TLS RFC 的 HTTP](https://www.ietf.org/rfc/rfc2818.txt)
- [HTTP 身份验证方案](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#authentication_schemesl)
- [Mozilla 网站安全建议](https://infosec.mozilla.org/guidelines/web_security)
- [Mozilla SSL 推荐的配置生成器](https://ssl-config.mozilla.org/)
- [Mozilla 服务器端 TLS 建议](https://wiki.mozilla.org/Security/Server_Side_TLS)
- [OpenSSH 主要手册页面](https://www.openssh.com/manual.html)
- [SSH RFC，详细介绍了可用于此协议的配置和架构](https://www.ietf.org/rfc/rfc4251.txt)
- [Mozilla OpenSSH 安全建议](https://infosec.mozilla.org/guidelines/openssh.html)
- [Android 密钥库系统](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)