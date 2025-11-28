---
title: Android-安全风险-已损坏或有风险的加密算法
date: 2025-11-28 11:25:25 +0800
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
description: 使用安全性较差或已损坏的加密函数（例如 `DES` 或 `RC4`）会对敏感数据的保密性造成严重风险。
math: true
---

# 已损坏或有风险的加密算法

**OWASP 类别**： [MASVS-CRYPTO：加密](https://mas.owasp.org/MASVS/06-MASVS-CRYPTO)

## 概览

尽管加密技术广泛用于保护数据机密性和完整性，但如果开发者无意中实现弱加密算法或过时的加密算法，就会带来重大风险。此漏洞源于这些算法的固有弱点，拥有必要计算能力或知识的恶意行为者可以利用这些弱点。此类漏洞利用的后果可能非常严重，可能会导致未经授权的访问、数据泄露和敏感信息遭操纵。

## 影响

敏感数据可能会被泄露、修改或伪造。存在缺陷或风险的加密算法可能会导致出现漏洞，并可能被滥用以解密敏感信息、篡改数据或冒充合法实体。利用此类漏洞的影响可能包括数据泄露、财务损失、声誉受损和用户信任度下降。

## 风险：加密哈希函数较弱或已损坏

使用弱加密哈希函数或已破解的加密哈希函数（例如 `MD5` 或 `SHA1`）会对数据的安全性和完整性造成严重风险。哈希函数旨在为输入数据创建唯一的固定长度指纹（哈希），因此可用于各种用途，包括数据完整性验证、密码存储和数字签名。不过，如果使用弱哈希函数或遭入侵的哈希函数，可能会出现以下几种漏洞：

- **碰撞攻击**：弱哈希函数容易受到碰撞攻击，攻击者会找到两个不同的输入，这两个输入会生成相同的哈希值。这样一来，他们就可以在不被发现的情况下用恶意数据替换合法数据，从而损害数据完整性。
- **数据泄露**：如果密码采用安全系数低的算法进行哈希处理，那么系统一旦被成功入侵，用户凭据就可能会泄露。然后，攻击者可能会使用彩虹表或其他技术破解密码，从而未经授权访问账号。
- **否认数字签名**：数字签名中使用的哈希函数如果安全系数较低，可能会被利用来创建伪造的签名，从而难以确定文档或消息的真实性和完整性。

### 缓解措施

为降低这些风险，务必使用经过充分检验的强加密哈希函数（例如 [`SHA-2`](https://en.wikipedia.org/wiki/SHA-2) 或 [`SHA-3`](https://en.wikipedia.org/wiki/SHA-3)），并在发现新漏洞时及时更新这些函数。此外，采用密码加盐和使用 [`bcrypt`](https://en.wikipedia.org/wiki/Bcrypt) 或 [`Argon2`](https://en.wikipedia.org/wiki/Argon2) 等密码专用哈希算法等安全实践，可以进一步增强数据保护。

---

## 风险：加密功能较弱或已损坏

使用安全性较差或已损坏的加密函数（例如 `DES` 或 `RC4`）会对敏感数据的保密性造成严重风险。加密旨在通过将信息转换为无法读取的格式来保护信息，但如果加密算法存在缺陷，这些保护措施可能会被绕过：

- **数据泄露**：加密算法较弱容易受到各种攻击，包括暴力攻击、已知明文攻击和密码分析技术。如果攻击成功，这些攻击可能会暴露加密数据，从而导致未经授权的方访问个人详细信息、财务记录或机密业务数据等敏感信息。
- **数据操纵和篡改**：即使攻击者无法完全解密数据，如果加密算法较弱，他们仍可能在不被发现的情况下操纵数据。这可能会导致数据遭到未经授权的修改，从而可能导致欺诈、虚假陈述或其他恶意活动。

### 缓解措施

#### 在加密函数中使用强加密算法

为降低这些风险，务必使用经过严格审核的强效加密算法，并遵循有关密钥管理和加密实现的最佳实践。定期更新加密算法并及时了解新出现的威胁，对于保持强大的数据安全性也至关重要。

以下是一些建议使用的默认算法：

- 对称加密：
    - `AES-128`/`AES-256`（采用 [`GCM`](https://en.wikipedia.org/wiki/Galois/Counter_Mode) 模式）
    - `Chacha20`
- 非对称加密：
    - `RSA-2048`/`RSA-4096`，内边距为 [`OAEP`](https://datatracker.ietf.org/doc/html/rfc8017)

#### 使用加密库中的安全原语来减少常见陷阱

虽然选择合适的加密算法至关重要，但为了真正最大限度地减少安全漏洞，请考虑使用提供简化的 API 并强调安全默认配置的加密库。这种方法不仅可以增强应用的安全性，还可以显著降低因编码错误而引入漏洞的可能性。例如，[Tink](https://developers.google.com/tink?hl=zh-cn) 通过提供两种截然不同的选项（[`AEAD`](https://developers.google.com/tink/streaming-aead?hl=zh-cn) 和 [`Hybrid`](https://developers.google.com/tink/hybrid?hl=zh-cn) 加密）来简化加密选择，从而让开发者更轻松地做出明智的安全决策。

---

## 风险：加密签名功能较弱或损坏

使用弱加密签名函数或已损坏的加密签名函数（例如 [`RSA-PKCS#1 v1.5`](https://www.rfc-editor.org/rfc/rfc2313) 或基于弱哈希函数的签名函数）会对数据和通信的完整性造成严重风险。数字签名旨在提供身份验证、不可否认性和数据完整性，确保消息或文档来自特定发送者，并且未被篡改。不过，如果底层签名算法存在缺陷，这些保证可能会受到影响：

- **伪造签名**：弱签名算法可能容易受到攻击，从而让恶意行为者能够创建伪造的签名。这意味着，他们可以冒充合法实体、伪造文档或篡改消息，而不会被发现。
- **签名否认**：如果签名算法被破解，签名者可能会谎称自己未签署某份文档，从而破坏不可否认原则，并造成法律和后勤方面的难题。
- **数据操纵和篡改**：在签名用于保护数据完整性的场景中，如果算法较弱，攻击者可能会在不使签名失效的情况下修改数据，从而导致篡改行为无法被检测到，并可能导致关键信息泄露。

### 缓解措施

#### 使用强大的加密签名算法

为降低这些风险，务必要使用经过严格审核的强加密签名算法：

- `RSA-2048`/`RSA-4096`，内边距为 [`PSS`](https://datatracker.ietf.org/doc/html/rfc8017)
- 采用安全曲线的椭圆曲线数字签名算法 ([`ECDSA`](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm))

#### 使用加密库中的安全原语来减少常见陷阱

选择合适的签名算法至关重要，但为了真正最大限度地减少安全漏洞，请考虑使用默认情况下可提供可靠安全保证的加密库。例如，[Tink](https://developers.google.com/tink?hl=zh-cn) 通过提供以安全曲线为默认选项的 `ECDSA` 来简化签名选择，所有这些都在一个简单而全面的 API 中完成。这种方法不仅可以提高安全性，还可以省去复杂的配置或决策，从而简化开发流程。

---

## 资源

- [Tink 加密库](https://developers.google.com/tink/what-is?hl=zh-cn)
- [Android 应用质量：加密](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn)
- [使用 Tink 进行数字签名](https://developers.google.com/tink/digital-signature?hl=zh-cn)
- [使用 Tink 进行混合加密](https://developers.google.com/tink/hybrid?hl=zh-cn)
- [使用 Tink 进行身份验证加密](https://developers.google.com/tink/streaming-aead?hl=zh-cn)
- [弱加密哈希函数和加密函数或已损坏的加密哈希函数和加密函数 Android 安全 lint](https://github.com/google/android-security-lints/blob/main/checks/src/main/java/com/example/lint/checks/BadCryptographyUsageDetector.kt)
- [CWE-327：使用已损坏或有风险的加密算法](https://cwe.mitre.org/data/definitions/327.html)
- [CWE-328：使用弱哈希](https://cwe.mitre.org/data/definitions/328.html)
- [CWE-780：未使用 OAEP 的 RSA 算法](https://cwe.mitre.org/data/definitions/780.html)
- [NIST 关于已获批准的哈希函数的页面](https://csrc.nist.gov/projects/hash-functions)
- [高级加密标准（维基百科）](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
- [Secure Hash Algorithm 2（维基百科）](https://en.wikipedia.org/wiki/SHA-2)
- [Secure Hash Algorithm 3（维基百科）](https://en.wikipedia.org/wiki/SHA-3)
- [RSA 加密系统（维基百科）](https://en.wikipedia.org/wiki/RSA_\(cryptosystem\))
- [椭圆曲线数字签名算法（维基百科）](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
- [Stream cipher ChaCha（维基百科）](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant)
- [密码加盐（维基百科）](https://en.wikipedia.org/wiki/Salt_\(cryptography\))
- [混合密码系统（维基百科）](https://en.wikipedia.org/wiki/Hybrid_cryptosystem)
- [经过身份验证的加密](https://en.wikipedia.org/wiki/Authenticated_encryption)
