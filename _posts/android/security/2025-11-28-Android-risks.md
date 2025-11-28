---
title: Android-常见风险
date: 2025-11-28 10:43:29 +0800
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
description: 提高应用程序的安全性，以维护用户信任、保护设备完整性并保障您的数据安全。
math: true
---
# 降低应用的安全风险

提高应用的安全性有助于维护用户信任和设备完整性。

本页将介绍 Android 应用开发者面临的一系列常见安全问题。您可以通过以下方式使用这些内容：

- 详细了解如何主动保护应用安全。
- 了解如果在您的应用中发现了其中某个问题，您该如何应对。

以下列表包含各个问题对应的专门页面的链接，依 [OWASP MASVS](https://mas.owasp.org/MASVS/) 控制项分门别类。每个页面都包含摘要、影响声明和关于如何降低应用风险的提示。

### MASVS-STORAGE：存储

[OWASP 类别说明](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

移动应用程序处理各种敏感数据，例如个人身份信息 (PII)、加密材料、密钥和 API 密钥，这些数据通常需要存储在本地。这些敏感数据可能存储在私密位置（例如应用程序的内部存储）或用户或设备上安装的其他应用程序可以访问的公共文件夹中。但是，敏感数据也可能在无意中存储或暴露到公共位置，这通常是使用某些 API 或系统功能（例如备份或日志）的副作用。

此类别旨在帮助开发者确保应用程序有意存储的任何敏感数据都能得到妥善保护，无论目标位置如何。它还涵盖因不当使用 API 或系统功能而可能发生的意外泄露。

|ID|陈述|
|---|---|
|[MASVS-STORAGE-1](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-1)|该应用程序可安全地存储敏感数据。|
|[MASVS-STORAGE-2](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-2)|该应用程序可防止敏感数据泄露。|

- [备份泄漏](https://developer.android.com/topic/security/risks/backup-leaks?hl=zh-cn)
- [向 FileProvider 不当暴露目录](https://developer.android.com/topic/security/risks/file-providers?hl=zh-cn)
- [日志信息披露](https://developer.android.com/topic/security/risks/log-info-disclosure?hl=zh-cn)
- [路径遍历](https://developer.android.com/topic/security/risks/path-traversal?hl=zh-cn)
- [压缩路径遍历](https://developer.android.com/topic/security/risks/zip-path-traversal?hl=zh-cn)

### MASVS-CRYPTO：加密

[OWASP 类别说明](https://mas.owasp.org/MASVS/06-MASVS-CRYPTO)

加密技术对于移动应用至关重要，因为移动设备便携性强，容易丢失或被盗。这意味着，攻击者一旦获得设备的物理访问权限，就有可能访问设备上存储的所有敏感数据，包括密码、财务信息和个人身份信息。加密技术通过加密来保护这些敏感数据，使其不易被未经授权的用户读取或访问。

此类别中的控制措施旨在确保经过验证的应用程序按照行业最佳实践使用加密技术，这些最佳实践通常在外部标准（例如[NIST.SP.800-175B ↗](https://csrc.nist.gov/publications/detail/sp/800-175b/rev-1/final)和[NIST.SP.800-57 ↗](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final) ）中定义。此类别还侧重于加密密钥在其整个生命周期中的管理，包括密钥生成、存储和保护。糟糕的密钥管理甚至会危及最强大的加密技术，因此开发人员必须遵循推荐的最佳实践，以确保用户敏感数据的安全。

|ID|陈述|
|---|---|
|[MASVS-CRYPTO-1](https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-1)|该应用程序采用当前强大的加密技术，并按照行业最佳实践进行使用。|
|[MASVS-CRYPTO-2](https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-2)|该应用程序按照行业最佳实践执行密钥管理。|

- [硬编码的加密密钥](https://developer.android.com/topic/security/risks/hardcoded-cryptographic-secrets?hl=zh-cn)
- [弱 PRNG](https://developer.android.com/topic/security/risks/weak-prng?hl=zh-cn)

### MASVS-NETWORK：网络通信

[OWASP 类别说明](https://mas.owasp.org/MASVS/08-MASVS-NETWORK)

安全的网络连接是移动应用安全的关键环节，尤其对于那些通过网络通信的应用而言更是如此。为了确保传输中数据的机密性和完整性，开发者通常依赖于远程端点的加密和身份验证，例如使用TLS协议。然而，开发者可能通过多种方式意外禁用平台默认的安全机制，或者通过使用底层API或第三方库完全绕过这些机制。

此类别旨在确保移动应用在任何情况下都能建立安全连接。具体而言，它侧重于验证应用是否建立了安全加密的网络通信通道。此外，此类别还涵盖开发者可能选择仅信任特定证书颁发机构 (CA) 的情况，这通常被称为证书绑定或公钥绑定。

|ID|陈述|
|---|---|
|[MASVS-NETWORK-1](https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-1)|该应用程序根据当前最佳实践保护所有网络流量。|
|[MASVS-NETWORK-2](https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-2)|该应用程序对开发者控制下的所有远程端点执行身份绑定。|

- [明文/纯文本 HTTP](https://developer.android.com/topic/security/risks/cleartext?hl=zh-cn)

### MASVS-PLATFORM：平台互动

[OWASP 类别说明](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

移动应用的安全性很大程度上取决于它们与移动平台的交互，而这种交互通常涉及通过使用平台提供的进程间通信 (IPC) 机制和 WebView 等技术有意地暴露数据或功能，以提升用户体验。然而，这些机制也可能被攻击者或其他已安装的应用利用，从而危及应用的安全。

此外，敏感数据，例如密码、信用卡信息和通知中的一次性密码，通常会显示在应用程序的用户界面中。因此，必须确保这些数据不会通过平台机制（例如自动生成的屏幕截图）或因他人偷窥或设备共享等意外方式泄露。

此类别包含确保应用程序与移动平台交互安全的控制措施。这些控制措施涵盖平台提供的进程间通信 (IPC) 机制的安全使用、防止敏感数据泄露和功能暴露的 WebView 配置，以及在应用程序用户界面中安全显示敏感数据。通过实施这些控制措施，移动应用程序开发人员可以保护敏感用户信息，并防止攻击者未经授权的访问。

|ID|陈述|
|---|---|
|[MASVS-PLATFORM-1](https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-1)|该应用程序安全地使用了进程间通信（IPC）机制。|
|[MASVS-PLATFORM-2](https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-2)|该应用安全地使用了WebView。|
|[MASVS-PLATFORM-3](https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-3)|该应用程序使用安全的用户界面。|

- [内容解析器](https://developer.android.com/topic/security/risks/content-resolver?hl=zh-cn)
- [隐式 intent 盗用](https://developer.android.com/topic/security/risks/implicit-intent-hijacking?hl=zh-cn)
- [intent 重定向](https://developer.android.com/topic/security/risks/intent-redirection?hl=zh-cn)
- [待处理 intent](https://developer.android.com/topic/security/risks/pending-intent?hl=zh-cn)
- [粘性广播](https://developer.android.com/topic/security/risks/sticky-broadcast?hl=zh-cn)
- [StrandHogg 攻击/任务相关性漏洞](https://developer.android.com/topic/security/risks/strandhogg?hl=zh-cn)
- [点按劫持](https://developer.android.com/topic/security/risks/tapjacking?hl=zh-cn)
- [android:debuggable](https://rd-wang.github.io/posts/Android-risks-android-debuggable/)
- [android:exported](https://rd-wang.github.io/posts/Android-risks-android-exported/)

### MASVS-CODE：代码质量

[OWASP 类别说明](https://mas.owasp.org/MASVS/10-MASVS-CODE)

移动应用拥有众多数据入口点，包括用户界面 (UI)、进程间通信 (IPC)、网络和文件系统，这些入口点可能会接收到被不受信任的攻击者无意中修改的数据。通过将这些数据视为不受信任的输入，并在使用前对其进行适当的验证和清理，开发人员可以防止诸如 SQL 注入、跨站脚本攻击 (XSS) 或不安全的反序列化等经典注入攻击。然而，其他常见的编码漏洞，例如内存损坏缺陷，虽然难以通过渗透测试检测到，但可以通过安全的架构和编码实践轻松预防。开发人员应遵循[OWASP 软件保障成熟度模型 (SAMM) ↗](https://owaspsamm.org/model/)和[NIST SP 800-218 安全软件开发框架 (SSDF) ↗](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)等最佳实践，从源头上避免引入这些缺陷。

此类别涵盖源自外部来源的代码漏洞，例如应用程序数据入口点、操作系统和第三方软件组件。开发人员应验证并清理所有传入数据，以防止注入攻击和绕过安全检查。他们还应强制执行应用程序更新，并确保应用程序运行在最新的平台上，以保护用户免受已知漏洞的侵害。

|ID|陈述|
|---|---|
|[MASVS-CODE-1](https://mas.owasp.org/MASVS/controls/MASVS-CODE-1)|该应用需要最新版本的平台。|
|[MASVS-CODE-2](https://mas.owasp.org/MASVS/controls/MASVS-CODE-2)|该应用程序具有强制更新应用程序的机制。|
|[MASVS-CODE-3](https://mas.owasp.org/MASVS/controls/MASVS-CODE-3)|该应用程序仅使用不存在已知漏洞的软件组件。|
|[MASVS-CODE-4](https://mas.owasp.org/MASVS/controls/MASVS-CODE-4)|该应用程序会对所有不受信任的输入进行验证和清理。|

- [不安全的 API 或库](https://developer.android.com/topic/security/risks/insecure-library?hl=zh-cn)
- [SQL 注入](https://developer.android.com/topic/security/risks/sql-injection?hl=zh-cn)
- [测试/调试功能](https://developer.android.com/topic/security/risks/test-debug?hl=zh-cn)
- [不安全的 HostnameVerifier](https://developer.android.com/topic/security/risks/unsafe-hostname?hl=zh-cn)
- [WebView：不安全的 URI 加载](https://developer.android.com/topic/security/risks/unsafe-uri-loading?hl=zh-cn)

