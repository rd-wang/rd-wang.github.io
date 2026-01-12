---
title: Android-安全性-安全风险-不安全的API使用方式
date: 2025-12-1 08:11:07 +0800
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
description: 攻击者窃取密钥，利用此访问权限进行欺诈、重定向服务，在极少数情况下，甚至可以完全控制服务器。
math: true
---

# 不安全的 API 使用方式

**OWASP 类别**： [MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

许多移动应用都使用外部 API 来提供功能。传统上，静态令牌或密钥用于对连接到服务的应用进行身份验证。

不过，在客户端-服务器设置（或移动应用和 API）的背景下，使用静态密钥通常不被认为是访问敏感数据或服务的安全身份验证方法。与内部基础设施不同，如果有人有权访问此密钥，则可以访问外部 API 并滥用该服务。

例如，静态密钥可能会被从应用中逆向工程出来，或者在移动应用与外部 API 通信时被拦截。此外，此静态密钥更有可能在应用中进行硬编码。

如果未使用足够安全的身份验证和访问控制措施，API 数据和服务就会面临风险。

使用静态密钥时，攻击者可以利用该 API 通过重放请求或使用（截获或逆向工程）密钥构造新请求来攻击该 API，而不会受到任何时间限制。

## 影响

如果开发者付费使用 AI 或 ML API 服务，攻击者相对容易窃取此密钥，从而导致开发者拖欠服务费用或利用该密钥创建虚假内容。

存储在 API 上的任何用户数据、文件或 PII 都将可供自由访问，从而导致破坏性泄露。

攻击者还可能会利用此访问权限进行欺诈、重定向服务，在极少数情况下，甚至可以完全控制服务器。

## 缓解措施

### 有状态 API

如果应用提供用户登录功能或能够跟踪用户会话，我们建议开发者使用 [Google Cloud](https://cloud.google.com/apis?hl=zh-cn) 等 API 服务，以便与应用进行有状态集成。

此外，请使用 API 服务提供的安全身份验证、客户端验证和会话控制，并考虑使用动态令牌来代替静态密钥。确保令牌在合理的时间内过期（通常为 1 小时）。

然后，应使用动态令牌进行身份验证，以提供对 API 的访问权限。[这些指南](https://developers.google.com/identity/protocols/oauth2/native-app?hl=zh-cn#android)介绍了如何使用 OAuth 2.0 实现此目的。除了这些准则之外，您还可以通过实现以下功能来进一步加强 OAuth 2.0，以减少 Android 应用中的漏洞。

实现适当的错误处理和日志记录：

- 妥善处理 OAuth 错误，以显眼的方式显示这些错误，并记录这些错误以供调试。
    - 缩小受攻击面还有助于您发现并排查可能出现的任何问题。
    - 确保记录或向用户显示的任何消息清晰明了，但不包含对攻击者有用的[信息](https://owasp.org/www-community/Improper_Error_Handling)。

在应用中安全地配置 OAuth：

- 将 `redirect_uri` 参数发送到授权端点和令牌端点。
- 如果您的应用将 OAuth 与第三方服务搭配使用，请配置跨源资源共享 ([CORS](https://owasp.org/www-community/attacks/CORS_OriginHeaderScrutiny))，以限制对应用资源的访问权限。
    - 这有助于防范未经授权的跨站脚本 ([XSS](https://owasp.org/www-community/attacks/xss/)) 攻击。
- 使用 state 参数可防止 [CSRF](https://owasp.org/www-community/attacks/csrf) 攻击。

使用安全库：

- 请考虑使用 [AppAuth](https://openid.github.io/AppAuth-Android/) 等安全库来简化安全 OAuth [流程](https://developers.google.com/identity/protocols/oauth2/native-app?hl=zh-cn)的实现。
    - 这些 Android 库有助于自动执行前面提到的许多安全最佳实践。

[OpenAPI 文档](https://cloud.google.com/endpoints/docs/openapi/authentication-method?hl=zh-cn)中介绍了其他身份验证方法，包括 Firebase 和 Google ID 令牌。

### 无状态 API

如果 API 服务未提供上述任何建议的保护措施，并且需要无状态会话（无需用户登录），开发者可能需要提供自己的中间件解决方案。

这会涉及在应用和 API 端点之间“代理”请求。一种方法是使用 [JSON Web 令牌](https://en.wikipedia.org/wiki/JSON_Web_Token) (JWT) 和 [JSON Web 签名](https://en.wikipedia.org/wiki/JSON_Web_Signature) (JWS)，或者提供完全经过身份验证的服务（如前所述）。这样一来，便可在服务器端而非应用（客户端）中存储易受攻击的静态密钥。

在移动应用中提供端到端无状态解决方案存在固有的问题。其中一些最关键的挑战包括验证客户端（应用）以及在设备上安全地存储私钥或密钥。

[Play Integrity API](https://developer.android.com/google/play/integrity/overview?hl=zh-cn) 可验证应用和请求的完整性。这可以缓解滥用此访问权限的一些情况。至于密钥管理，在许多情况下，[密钥库](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)是安全存储私钥的最佳位置。

部分移动应用使用注册阶段来检查应用的完整性，并通过安全交换提供密钥。这些方法很复杂，不在本文档的讨论范围内。不过，[云密钥管理服务](https://cloud.google.com/kms/docs/key-management-service?hl=zh-cn)是一种潜在的解决方案。

## 资源

- [OAuth 2.0 建议](https://developers.google.com/identity/protocols/oauth2/native-app?hl=zh-cn)
- [有关使用 OAuth 2.0 和 JWT 的建议](https://tech.justeattakeaway.com/2019/12/04/lessons-learned-from-handling-jwt-on-mobile/)
- [OAuth 客户端建议](https://portswigger.net/web-security/oauth/preventing)
