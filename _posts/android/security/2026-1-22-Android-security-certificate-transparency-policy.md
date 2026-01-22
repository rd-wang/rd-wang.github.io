---
title: Android-安全性-Android 证书透明度政策
date: 2026-1-22 15:47:05 +0800
categories:
  - Android
  - Security
  - Certificate 
  - Policy
tags:
  - Android
  - Security
  - Certificate
  - Policy
description: 当应用中验证连接的传输层安全性 (TLS) 证书时，如果该应用选择启用证书透明度，系统会根据 Android 证书透明度 (CT) 政策评估其合规性。
math: true
---
_如有关于本政策的任何问题，请发送电子邮件至 CT 政策论坛： [ct-policy@chromium.org](https://groups.google.com/a/chromium.org/forum/?hl=zh-cn#!forum/ct-policy)_

当应用中验证连接的传输层安全性 (TLS) 证书时，如果该应用[选择启用证书透明度](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#CertificateTransparencySummary)，系统会根据 Android 证书透明度 (CT) 政策评估其合规性。如果证书附带的签名证书时间戳 (SCT) 满足此政策，则称该证书符合 CT 标准。

CT 合规性是通过证书和一组随附的 SCT 来实现的，这些证书和随附的 SCT 满足在证书验证期间由流行的 TLS 库（包括内置的 Conscrypt）强制执行的一组技术要求，这些要求在本策略中定义。

## CT 日志状态

Android 中的 CT 合规性是通过评估 CT 日志中的 SCT 来确定的，并确保这些日志在检查时处于正确的状态。CT 日志可能处于的状态集如下：

- `Pending`
- `Qualified`
- `Usable`
- `ReadOnly`
- `Retired`
- `Rejected`

为了帮助您了解 Android 中与 CT 合规性相关的要求，Chrome 文档的 [CT 日志生命周期说明](https://googlechrome.github.io/CertificateTransparency/log_states.html)详细介绍了这些状态的定义、每种状态下日志的要求，以及这些状态对 Android 行为的影响。

## 符合 CT 标准的证书

如果 TLS 证书附带的一组 SCT 并且满足后面定义的至少一个标准（具体取决于 SCT 传递给 Android 的方式），则该 TLS 证书 _CT _合规_ 的。在 Android 的强制执行 CT 的应用中，所有公开信任的 TLS 证书都必须符合 CT 标准才能成功验证；不过，未记录到 CT 或具有足够 SCT 的证书不会被视为错误签发的证书。

在评估证书是否符合 CT 标准时，Android 会考虑多种因素，包括 SCT 的数量、签发 SCT 的 CT 日志的运营方，以及签发 SCT 的 CT 日志在证书验证时和 CT 日志创建 SCT 时的状态。

根据向 Android 呈现 SCT 的方式，满足以下条件之一即可实现 CT 合规性：

**嵌入式 SCT**：

1. 至少一个来自 CT 日志的嵌入式 SCT，该 CT 日志在检查时处于 `Qualified`、`Usable` 或 `ReadOnly` 状态；并且
2. 至少有 N 个不同的 CT 日志中的嵌入式 SCT 在检查时处于 `Qualified`、`Usable`、`ReadOnly` 或 `Retired` 状态，其中 N 的定义如下表所示；并且

| 证书有效期    | 来自不同 CT 日志的 SCT 数量 |
| -------- | ------------------ |
| <= 180 天 | 2                  |
| 超过 180 天 | 3                  |

3. 在满足要求 2 的 SCT 中，至少有两个 SCT 必须由 Android 认可的不同 CT 日志运营商签发；并且
4. 在满足要求 2 的 SCT 中，至少有一个 SCT 必须由 Android 识别为符合 [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962) 的日志签发。

**通过 OCSP 或 TLS 传送的 SCT**：

1. 在检查时 CT 日志中至少有两个 SCT 处于 `Qualified`、`Usable` 或 `ReadOnly` 状态；并且
2. 在满足要求 1 的 SCT 中，至少有两个 SCT 必须由 Android 认可的不同 CT 日志运营商签发；并且
3. 在满足要求 1 的 SCT 中，至少有一个 SCT 必须由 Android 认可的符合 RFC6962 标准的 CT 日志签发。

对于嵌入式 SCT 和通过 OCSP 或 TLS 传送的 SCT，日志运算符的唯一性是指在[日志列表](https://developer.android.com/privacy-and-security/certificate-transparency-policy?hl=zh-cn#log-list)的运算符部分中具有单独的条目。

在 CT 日志在其生命周期内发生运营商变更的罕见情况下，CT 日志可以选择包含一个 `previous_operators`  列表，以及该日志由前一个运营商操作的最后时间戳。为防止日志运营商变更导致现有证书失效，通过将 SCT 时间戳与 `previous_operators `时间戳（如果存在）进行比较，确定每个 SCT 的日志操作员是 SCT 颁发时的运营商。

**重要注意事项**

只要握手中呈现的 SCT 的某种组合满足上述某个 CT 合规性条件，那么无论 SCT 的状态如何，额外的 SCT 都不会对证书的 CT 合规性状态产生正面或负面影响。

为了有助于证书符合 CT 标准，SCT 必须在日志的 `Retired` 时间戳（如果存在）之前签发。Android 会使用所有提供的 SCT 中最早的 SCT 来根据 CT 日志 `Retired` 时间戳评估 CT 合规性。这考虑了在提交证书日志记录请求的过程中，CT 日志变为“已停用”的极端情况。

“嵌入式 SCT”是指使用证书本身的 `SignedCertificateTimestampList` X.509v3  扩展提供的 SCT。许多 TLS 服务器不支持 OCSP Stapling 或 TLS 扩展，因此 CA 应准备好将 SCT 嵌入到已签发的证书中，以确保在 Android 中成功进行验证或 EV 处理。

## 如何将 CT 日志添加到 Android

如需了解 CT 日志如何成为 `Qualified` 以及在哪些情况下会成为 `Retired`，请参阅 Chrome CT [日志政策](https://googlechrome.github.io/CertificateTransparency/log_policy.html)。

## CT 强制执行超时时间

Google 每天都会发布包含最新`log_list_timestamp`的新 CT 日志列表。Android 设备每天会尝试下载此列表的最新版本以进行验证。在任何给定时间，如果设备上没有可用的日志列表，或者日志列表的时间戳超过 70 天，则会停用 CT 强制执行。 此超时时间为 CT 生态系统提供了一项关键保证，即新的 CT 日志在变为 `Qualified` 后，能够在固定时间内安全过渡到“可用”状态。

## Android 日志列表

Android 日志列表发布在 [log_list.json](https://www.gstatic.com/android/certificate_transparency/v2/log_list.json) 中，每天更新一次。 此日志列表不提供稳定的 API、SLA 或可用性保证。

Android 会提供其 CT 日志列表，以供证书提交者（例如认证机构）以及希望与 CT 和 WebPKI 生态系统保持兼容或调查其内容的 CT 监控器和审核员使用。

> **注意**： 未经 Android CT 团队明确书面许可，不得使用 Android CT 日志列表在 Android 平台本身以外的 TLS 客户端中强制执行 CT。

未经授权就依赖 Android CT 日志列表不仅会危及您的用户，还会危及 Android 用户和整个 CT 生态系统。如果您正在探索如何将 CT 强制执行添加到应用中，请使用 [Android 平台支持的机制](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#CertificateTransparencySummary)。

违反此策略使用 Android CT 日志列表的风险由您自行承担，并可能导致您的应用程序或库出现故障。Android 必须能够根据 CT 生态系统中的事件对 CT 日志列表进行更改，以维护 Android 用户的安全。Android 可能会采取措施，确保第三方对 CT 日志列表的依赖不会危及 Android 应对此类事件的能力，包括对日志列表进行未公开的更改以阻止未经授权的使用。

