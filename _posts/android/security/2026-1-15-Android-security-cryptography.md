---
title: Android-安全性-加密
date: 2026-1-15 15:56:17 +0800
categories:
  - Android
  - Security
  - cryptography
tags:
  - Android
  - Security
  - cryptography
description: 介绍使用 Android 加密工具的正确方法
math: true
---
本文档将介绍使用 Android 加密工具的正确方法，并提供一些使用示例。如果您的应用需要更高的密钥安全性，请使用 [Android 密钥库系统](https://developer.android.com/training/articles/keystore?hl=zh-cn)。

> **注意** ：除非另有说明，否则本文中的建议适用于所有 Android 版本。

## 仅在使用 Android 密钥库系统时才指定提供程序

如果使用 Android 密钥库系统，则**必须**指定提供程序。

而在其他情况下，Android 并不保证为指定算法提供特定的提供程序。如果在未使用 Android 密钥库系统的情况下指定提供程序，则可能会导致未来版本出现兼容性问题。

## 选择建议的算法

如果可以自由选择使用哪种算法（例如在不需要与第三方系统兼容时），建议您使用以下算法：

| 类             | 建议                                                      |
| ------------- | ------------------------------------------------------- |
| Cipher        | 采用 CBC 或 GCM 模式且具有 256 位密钥的 AES（例如 `AES/GCM/NoPadding`） |
| MessageDigest | SHA-2 系列（例如 `SHA-256`）                                  |
| Mac           | SHA-2 系列 HMAC（例如 `HMACSHA256`）                          |
| 签名            | 使用 ECDSA 的 SHA-2 系列（例如 `SHA256withECDSA`）               |

> **注意**：在读取和写入本地文件时，您的应用可以使用 [Security 库](https://developer.android.com/topic/security/data?hl=zh-cn)以更安全的方式执行这些操作。该库会指定一个建议的加密算法。

## 执行常见的加密操作

下面几部分提供的代码段演示了如何在应用中完成常见的加密操作。

### 加密消息

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val plaintext: ByteArray = ...
val keygen = KeyGenerator.getInstance("AES")
keygen.init(256)
val key: SecretKey = keygen.generateKey()
val cipher = Cipher.getInstance("AES/CBC/PKCS5PADDING")
cipher.init(Cipher.ENCRYPT_MODE, key)
val ciphertext: ByteArray = cipher.doFinal(plaintext)
val iv: ByteArray = cipher.iv
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
byte[] plaintext = ...;
KeyGenerator keygen = KeyGenerator.getInstance("AES");
keygen.init(256);
SecretKey key = keygen.generateKey();
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5PADDING");
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] ciphertext = cipher.doFinal(plaintext);
byte[] iv = cipher.getIV();
```


### 生成消息摘要

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val message: ByteArray = ...
val md = MessageDigest.getInstance("SHA-256")
val digest: ByteArray = md.digest(message)
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
byte[] message = ...;
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] digest = md.digest(message);
```


### 生成数字签名

您需要拥有一个包含签名密钥的 [`PrivateKey`](https://developer.android.com/reference/java/security/PrivateKey?hl=zh-cn) 对象；该密钥可以在运行时生成、从与您的应用捆绑在一起的文件中读取，或者根据您的需要从其他一些来源获取。

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val message: ByteArray = ...
val key: PrivateKey = ...
val s = Signature.getInstance("SHA256withECDSA")
        .apply {
            initSign(key)
            update(message)
        }
val signature: ByteArray = s.sign()
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
byte[] message = ...;
PrivateKey key = ...;
Signature s = Signature.getInstance("SHA256withECDSA");
s.initSign(key);
s.update(message);
byte[] signature = s.sign();
```


### 验证数字签名

您需要拥有一个包含签名者公钥的 [`PublicKey`](https://developer.android.com/reference/kotlin/java/security/PublicKey?hl=zh-cn) 对象；该公钥可以从与您的应用捆绑在一起的文件中读取、[从证书中提取](https://developer.android.com/reference/javax/security/cert/Certificate?hl=zh-cn#getPublicKey\(\))，或者根据您的需要从其他一些来源获取。

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val message: ByteArray = ...
val signature: ByteArray = ...
val key: PublicKey = ...
val s = Signature.getInstance("SHA256withECDSA")
        .apply {
            initVerify(key)
            update(message)
        }
val valid: Boolean = s.verify(signature)
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
byte[] message = ...;
byte[] signature = ...;
PublicKey key = ...;
Signature s = Signature.getInstance("SHA256withECDSA");
s.initVerify(key);
s.update(message);
boolean valid = s.verify(signature);
```
## 实现方面的复杂问题

Android 加密实现的一些细节看似不寻常，但因兼容性方面的考虑而存在。本部分探讨了您最有可能遇到的一些细节。

### OAEP MGF1 消息摘要

RSA OAEP 加密算法由以下两个不同的消息摘要进行参数化：“主”摘要和 MGF1 摘要。有些 [`Cipher`](https://developer.android.com/reference/javax/crypto/Cipher?hl=zh-cn) 标识符包含摘要名，例如 `Cipher.getInstance("RSA/ECB/OAEPwithSHA-256andMGF1Padding")`（该标识符指定了主摘要，而未指定 MGF1 摘要）。在 Android 密钥库中，SHA-1 用于 MGF1 摘要；而在其他 Android 加密提供程序中，这两个摘要相同。

为了更好地控制您的应用使用的摘要，您应该请求带有 OAEPPadding 的加密算法（像 `Cipher.getInstance("RSA/ECB/OAEPPadding")` 一样），并向 `init()` 提供 `OAEPParameterSpec` 以明确选择这两个摘要。如以下代码所示：

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val key: Key = ...
val cipher = Cipher.getInstance("RSA/ECB/OAEPPadding")
        .apply {
            // To use SHA-256 the main digest and SHA-1 as the MGF1 digest
            init(Cipher.ENCRYPT_MODE, key, OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA1, PSource.PSpecified.DEFAULT))
            // To use SHA-256 for both digests
            init(Cipher.ENCRYPT_MODE, key, OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA256, PSource.PSpecified.DEFAULT))
        }
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
Key key = ...;
Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPPadding");
// To use SHA-256 the main digest and SHA-1 as the MGF1 digest
cipher.init(Cipher.ENCRYPT_MODE, key, new OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA1, PSource.PSpecified.DEFAULT));
// To use SHA-256 for both digests
cipher.init(Cipher.ENCRYPT_MODE, key, new OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA256, PSource.PSpecified.DEFAULT));
```


## 已废弃的功能

以下几个部分介绍的是已废弃的功能。请勿在应用中使用该功能。

### Bouncy Castle 算法

许多算法的 [Bouncy Castle](https://www.bouncycastle.org/) 实现都[已废弃](https://android-developers.googleblog.com/2018/03/cryptography-changes-in-android-p.html)。这只会影响您明确请求了 Bouncy Castle 提供程序的情况，如以下示例所示：

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
Cipher.getInstance("AES/CBC/PKCS7PADDING", "BC")
// OR
Cipher.getInstance("AES/CBC/PKCS7PADDING", Security.getProvider("BC"))
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
Cipher.getInstance("AES/CBC/PKCS7PADDING", "BC");
// OR
Cipher.getInstance("AES/CBC/PKCS7PADDING", Security.getProvider("BC"));
```

如[仅在使用 Android 密钥库系统时才指定提供程序](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#provider-android-keystore)部分中所述，不建议请求特定的提供程序。如果您遵循该准则，此废弃将不会对您产生影响。

### 不使用初始化向量的基于密码的加密算法

需要初始化向量 (IV) 的基于密码的加密 (PBE) 算法，可以从经过适当构造的密钥或者从明确传递的 IV 获得该向量。如果传递的 PBE 密钥不包含 IV 且未明确传递 IV，Android 上的 PBE 加密算法目前会假定 IV 为零。

使用 PBE 加密算法时，请务必明确传递 IV，如以下代码段所示：

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
val key: SecretKey = ...
val cipher = Cipher.getInstance("PBEWITHSHA256AND256BITAES-CBC-BC")
val iv = ByteArray(16)
SecureRandom().nextBytes(iv)
cipher.init(Cipher.ENCRYPT_MODE, key, IvParameterSpec(iv))
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
SecretKey key = ...;
Cipher cipher = Cipher.getInstance("PBEWITHSHA256AND256BITAES-CBC-BC");
byte[] iv = new byte[16];
new SecureRandom().nextBytes(iv);
cipher.init(Cipher.ENCRYPT_MODE, key, new IvParameterSpec(iv));
```

### Crypto 提供程序

自 Android 9（API 级别 28）起，Crypto Java 加密架构 (JCA) 提供程序已被移除。如果您的应用请求 Crypto 提供程序实例（例如通过调用以下方法来请求），则会出现 [`NoSuchProviderException`](https://developer.android.com/reference/java/security/NoSuchProviderException?hl=zh-cn)。

[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
SecureRandom.getInstance("SHA1PRNG", "Crypto")
```
[Java](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#java)
```java
SecureRandom.getInstance("SHA1PRNG", "Crypto");
```


### Jetpack 安全加密库

Jetpack 安全加密库已被废弃。这只会影响应用模块的 `build.gradle` 文件中存在以下依赖项的情况：

[Groovy](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#groovy)
```groovy
dependencies {
    implementation "androidx.security:security-crypto:1.1.0"
    // or
    implementation "androidx.security:security-crypto-ktx:1.1.0"
}
```
[Kotlin](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn#kotlin)
```kotlin
dependencies {
    implementation("androidx.security:security-crypto:1.1.0")
    // or
    implementation("androidx.security:security-crypto-ktx:1.1.0")
}
```

## 支持的算法

以下是 Android 上支持的 JCA 算法标识符：

- [`AlgorithmParameterGenerator`](https://developer.android.com/reference/java/security/AlgorithmParameterGenerator?hl=zh-cn)
- [`AlgorithmParameters`](https://developer.android.com/reference/java/security/AlgorithmParameters?hl=zh-cn)
- [`CertPathBuilder`](https://developer.android.com/reference/java/security/cert/CertPathBuilder?hl=zh-cn)
- [`CertPathValidator`](https://developer.android.com/reference/java/security/cert/CertPathValidator?hl=zh-cn)
- [`CertStore`](https://developer.android.com/reference/java/security/cert/CertStore?hl=zh-cn)
- [`CertificateFactory`](https://developer.android.com/reference/java/security/cert/CertificateFactory?hl=zh-cn)
- [`Cipher`](https://developer.android.com/reference/kotlin/javax/crypto/Cipher?hl=zh-cn)
- [`KeyAgreement`](https://developer.android.com/reference/kotlin/javax/crypto/KeyAgreement?hl=zh-cn)
- [`KeyFactory`](https://developer.android.com/reference/java/security/KeyFactory?hl=zh-cn)
- [`KeyGenerator`](https://developer.android.com/reference/kotlin/javax/crypto/KeyGenerator?hl=zh-cn)
- [`KeyManagerFactory`](https://developer.android.com/reference/javax/net/ssl/KeyManagerFactory?hl=zh-cn)
- [`KeyPairGenerator`](https://developer.android.com/reference/java/security/KeyPairGenerator?hl=zh-cn)
- [`KeyStore`](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn)
- [`Mac`](https://developer.android.com/reference/kotlin/javax/crypto/Mac?hl=zh-cn)
- [`MessageDigest`](https://developer.android.com/reference/java/security/MessageDigest?hl=zh-cn)
- [`SSLContext`](https://developer.android.com/reference/javax/net/ssl/SSLContext?hl=zh-cn)
- [`SSLEngine.Supported`](https://developer.android.com/reference/javax/net/ssl/SSLEngine?hl=zh-cn)
- [`SSLSocket.Supported`](https://developer.android.com/reference/javax/net/ssl/SSLSocket?hl=zh-cn)
- [`SecretKeyFactory`](https://developer.android.com/reference/kotlin/javax/crypto/SecretKeyFactory?hl=zh-cn)
- [`SecureRandom`](https://developer.android.com/reference/java/security/SecureRandom?hl=zh-cn)
- [`Signature`](https://developer.android.com/reference/java/security/Signature?hl=zh-cn)
- [`TrustManagerFactory`](https://developer.android.com/reference/javax/net/ssl/TrustManagerFactory?hl=zh-cn)