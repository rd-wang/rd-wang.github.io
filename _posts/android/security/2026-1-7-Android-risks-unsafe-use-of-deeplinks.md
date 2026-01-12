---
title: Android-安全性-安全风险-不安全地使用深层链接
date: 2026-1-7 10:15:29 +0800
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
description: 缺乏适当的深度链接验证机制，或不安全地使用深度链接，可能会帮助恶意用户实施诸如以下攻击：例如绕过主机验证、跨应用脚本攻击以及​​在易受攻击应用程序的权限上下文中执行远程代码。
math: true
---
**OWASP 类别**： [MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

深度链接带来的安全风险源于其核心功能，即在移动应用程序中实现无缝导航和交互。深度链接漏洞源于深度链接的实现或处理中的缺陷。恶意行为者可以利用这些缺陷来获取特权功能或数据，从而可能导致数据泄露、隐私侵犯和未经授权的行为。攻击者可以通过各种技术利用这些漏洞，例如深度链接劫持和数据验证攻击。

## 影响

缺乏适当的深度链接验证机制，或不安全地使用深度链接，可能会帮助恶意用户实施诸如以下攻击：例如绕过主机验证、跨应用脚本攻击以及​​在易受攻击应用程序的权限上下文中执行远程代码。根据应用程序的性质，这可能会导致对敏感数据或功能的未经授权访问。

## 缓解措施

### 防止深层链接被盗用

从设计上讲，Android 允许多个应用为同一深层链接 URI 注册 intent 过滤器。为防止恶意应用拦截指向您应用的深度链接，请在应用的 `AndroidManifest` 内的 `intent-filter` 中实现 `android:autoVerify` 属性。这样一来，用户可以选择自己喜欢的应用程序来处理深度链接，从而确保深度链接的预期操作，并防止恶意应用程序自动解析它们。

Android 12 [引入了](https://developer.android.com/about/versions/12/behavior-changes-all?hl=zh-cn#web-intent-resolution)更严格的 Web intent 处理方式，以提高安全性。现在，应用必须通过 Android App Links 或用户在系统设置中进行选择，才能处理来自特定域名的链接。这可以防止应用劫持它们不应该处理的链接。

如需为您的应用启用链接处理验证，请添加与以下格式匹配的 intent 过滤器（此示例摘自[验证 Android App Links](https://developer.android.com/training/app-links/verify-android-applinks?hl=zh-cn) 文档）：

```xml
  <!-- Make sure you explicitly set android:autoVerify to "true". -->
  <intent-filter android:autoVerify="true">
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
  
      <!-- If a user clicks on a shared link that uses the "http" scheme, your
           app should be able to delegate that traffic to "https". -->
      <data android:scheme="http" />
      <data android:scheme="https" />
  
      <!-- Include one or more domains that should be verified. -->
      <data android:host="..." />
  </intent-filter>
```

### 实现可靠的数据验证

深层链接可以包含提供给目标 intent 的其他参数，例如用于执行进一步操作。严格的数据验证是安全深层链接处理的基础。开发者应仔细验证和清理来自深层链接的所有传入数据，以防止恶意代码或值被注入合法应用中。这可以通过将任何深度链接参数的值与预定义的预期值列表进行比较来实现。

应用应先检查其他相关的内部状态（例如身份验证状态或授权），然后再公开敏感信息。例如，完成游戏关卡时获得的奖励。在这种情况下，值得验证已完成关卡这一前提条件，并在未完成关卡时重定向到主屏幕。

## 资源

- [验证 Android App Links](https://developer.android.com/training/app-links/verify-android-applinks?hl=zh-cn)
- [处理 Android App Links](https://developer.android.com/training/app-links?hl=zh-cn)
- [网络 intent 解析](https://developer.android.com/about/versions/12/behavior-changes-all?hl=zh-cn#web-intent-resolution)
- [拦截 Arrive 应用的魔法链接的账号盗用](https://hackerone.com/reports/855618)
- [深层链接和 WebView 入侵（第 I 部分）](https://www.justmobilesec.com/en/blog/deep-links-webviews-exploitations-part-I)
- [深层链接和 WebView 漏洞利用（第 II 部分）](https://www.justmobilesec.com/en/blog/deep-links-webviews-exploitations-part-II)
- [最近针对 Jetpack Navigation 中深层链接问题的建议](https://swarm.ptsecurity.com/android-jetpack-navigation-go-even-deeper/)

