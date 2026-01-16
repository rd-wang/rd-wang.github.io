---
title: Android-安全性-Android 密钥库系统
date: 2026-1-12 10:25:16 +0800
categories:
  - Android
  - Security
  - Keystore system
tags:
  - Android
  - Security
  - Keystore-System
description: 利用 Android 密钥库系统，保护密钥材料免遭未经授权的使用
math: true
---
Android Keystore 系统允许您将加密密钥存储在容器中，从而使其更难从设备中提取出来。在密钥进入密钥库后，可以将它们用于加密操作，而密钥材料不可导出。此外，密钥库系统允许您限制密钥的使用时间和方式，例如要求用户对密钥进行身份验证，或限制密钥只能在某些加密模式下使用。如需了解详情，请参阅[安全功能](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#SecurityFeatures)部分。

Keystore 系统由 Android 4.0（API 级别 14）中引入的 [KeyChain](https://developer.android.com/reference/android/security/KeyChain?hl=zh-cn) API 以及在 Android 4.3（API 级别 18）中引入的 Android Keystore 提供程序功能使用。本文介绍何时以及如何使用 Android 密钥库系统。

## 安全功能

Android 密钥库系统可以通过两种方式保护密钥材料免遭未经授权的使用。首先，它可以防止从应用进程和整个 Android 设备中提取密钥材料，降低了从 Android 设备外部未经授权使用密钥材料的风险。其次，密钥库系统通过让应用程序指定其密钥的授权用途，然后在应用程序进程之外强制执行这些限制，从而降低了在 Android 设备内未经授权使用密钥材料的风险。

### 提取防范

Android Keystore 密钥的关键材料通过两种安全措施防止被提取：

- 密钥材料永不进入应用进程。当应用程序使用 Android Keystore 密钥执行加密操作时，后台会将明文、密文以及要签名或验证的消息提供给执行加密操作的系统进程。如果应用程序进程遭到破坏，攻击者可能能够使用应用程序的密钥，但无法提取其密钥材料（例如，在 Android 设备之外使用）。
- 密钥材料可以绑定到 Android 设备的安全硬件（例如[Trusted Execution Environment 可信执行环境 (TEE)](https://source.android.com/docs/security/features/trusty?hl=zh-cn) 或[Secure Element 安全元件 (SE)](https://developer.android.com/privacy-and-security/keystore#strongbox_keymint_secure_element)）。启用此功能后，密钥的密钥材料将永远不会暴露在安全硬件之外。如果 Android 操作系统遭到入侵，或者攻击者可以读取设备的内部存储，攻击者可能能够使用 Android 设备上任何应用程序的 Android Keystore 密钥，但无法从设备中提取这些密钥。只有当设备的安全硬件支持密钥被授权使用的特定密钥算法、分块模式、填充方案和摘要组合时，此功能才能启用。
	 要检查某个密钥是否启用了该功能，请获取该密钥的 [KeyInfo](https://developer.android.com/reference/android/security/keystore/KeyInfo?hl=zh-cn)。下一步取决于您的应用的目标 SDK 版本：

	- 如果您的应用面向 Android 10（API 级别 29）或更高版本，请检查[getSecurityLevel()](https://developer.android.com/reference/android/security/keystore/KeyInfo?hl=zh-cn#getSecurityLevel\(\)) 的返回值。如果返回值与 `KeyProperties.SecurityLevelEnum.TRUSTED_ENVIRONMENT` 或 `KeyProperties.SecurityLevelEnum.STRONGBOX` 匹配，表示密钥位于安全硬件内。
    - 如果您的应用以 Android 9（API 级别 28）或更低版本为目标平台，请检查 [KeyInfo.isInsideSecurityHardware()](https://developer.android.com/reference/android/security/keystore/KeyInfo?hl=zh-cn#isInsideSecureHardware\(\)) 返回的布尔值。

### 硬件安全模块

运行 Android 9（API 级别 28）或更高版本的设备可以包含 [StrongBox KeyMint](https://source.android.com/docs/security/features/keystore)，它是 [StrongBox](https://source.android.com/docs/compatibility/15/android-15-cdd#9112_strongbox) 支持的 KeyMint HAL 的实现。虽然硬件安全模块（HSM）通常指的是能够抵御Linux内核攻击的安全密钥存储解决方案，StrongBox 特指嵌入式 SE 或集成安全隔离区 (iSE) 中的实现，与 TEE 相比，它能提供更强的隔离性和防篡改能力。

StrongBox KeyMint 的实现必须包含以下内容：

- 自己的 CPU
- 安全存储
- 真实随机数生成器
- 防止软件包篡改和未经授权的应用侧载的附加机制
- 安全计时器
- 重启通知引脚（或等效项），类似于通用输入/输出 (GPIO) 引脚

为了支持低能耗的 StrongBox 实现，仅支持部分算法和密钥长度：

- RSA 2048
- AES 128 和 256
- ECDSA、ECDH P-256
- HMAC-SHA256（支持 8-64 字节密钥大小，含首末值）
- Triple DES
- 扩展的长度 APDU

StrongBox 也支持[密钥认证](https://developer.android.com/privacy-and-security/security-key-attestation)。

> **注意：** StrongBox 适用于需要最高级别安全性的应用，特别是那些容易受到物理篡改或侧信道攻击的应用。然而，它的速度较慢，资源占用较高，并且支持的并发操作也较少。对于大多数应用程序而言，StrongBox 并非必需品。您应该仔细评估其增强的安全性是否符合您的应用程序的威胁模型和使用案例，因为它可能会带来性能上的权衡。

#### 使用 StrongBox KeyMint
使用 [`FEATURE_STRONGBOX_KEYSTORE`](https://developer.android.com/reference/kotlin/android/content/pm/PackageManager#feature_strongbox_keystore) 检查设备上是否可用 StrongBox。如果可用，您可以通过向以下方法传递 `true` 来指定将密钥存储在 StrongBox KeyMint 中的偏好：
- 密钥生成: [`KeyGenParameterSpec.Builder.setIsStrongBoxBacked()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder#setIsStrongBoxBacked\(boolean\))
- 密钥导入: [`KeyProtection.Builder.setIsStrongBoxBacked()`](https://developer.android.com/reference/android/security/keystore/KeyProtection.Builder#setIsStrongBoxBacked\(boolean\))

如果 StrongBox KeyMint 不支持指定的算法或密钥长度，框架将抛出​​ [`StrongBoxUnavailableException`](https://developer.android.com/reference/android/security/keystore/StrongBoxUnavailableException) 异常。如果发生这种情况，请生成或导入密钥，而无需调用 `setIsStrongBoxBacked(true)`.

### 密钥使用授权

为了防止未经授权使用 Android 设备上的密钥，Android Keystore 允许应用在生成或导入密钥时指定密钥的授权用途。密钥一旦生成或导入，其授权信息便无法更改。每次使用密钥时，Android 密钥库都会强制执行授权。这是一项高级安全功能，通常仅在以下情况下有用：您的应用程序进程在密钥生成/导入之后（而不是之前或期间）遭到破坏，不会导致密钥被未经授权使用。

支持的关键使用授权分为以下几类：

- **加密：** 密钥只能与授权的密钥算法、操作或用途（加密、解密、签名、验证）、填充方案、分组模式或摘要一起使用。
- **时间有效期：** 密钥仅在规定的时间段内有效。
- **用户身份验证：** 密钥仅在用户近期已通过身份验证的情况下才能使用。请参阅 [要求进行用户身份验证才能使用密钥](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#UserAuthentication)。

作为一项额外的安全措施，对于密钥材料位于安全硬件内的密钥（请参阅 [`KeyInfo.isInsideSecurityHardware()`](https://developer.android.com/reference/android/security/keystore/KeyInfo?hl=zh-cn#isInsideSecureHardware\(\))，或者对于面向 Android 10（API 级别 29）或更高版本的应用程序，请参阅 [`KeyInfo.getSecurityLevel()`](https://developer.android.com/reference/android/security/keystore/KeyInfo?hl=zh-cn#getSecurityLevel\(\))），根据安卓设备的不同，某些关键的使用授权可能由安全硬件强制执行。安全硬件通常会强制执行加密和用户身份验证授权。然而，安全硬件通常不会强制执行时间有效期授权，因为它通常没有独立的、安全的实时时钟。

您可以使用 [`KeyInfo.isUserAuthenticationRequirementEnforcedBySecureHardware()`](https://developer.android.com/reference/android/security/keystore/KeyInfo#isUserAuthenticationRequirementEnforcedBySecureHardware\(\))查询密钥的用户身份验证授权是否由安全硬件强制执行。

## 在钥匙串和 Android 密钥库提供商之间进行选择

当您需要系统级凭据时，请使用 [`KeyChain`](https://developer.android.com/reference/android/security/KeyChain) API。当应用通过 `KeyChain` API 请求使用任何凭据时，用户可以通过系统提供的用户界面选择应用可以访问哪些已安装的凭据。这样一来，在用户同意的情况下，多个应用程序可以使用同一组凭据。

使用 Android Keystore 提供程序，可以让单个应用程序存储自己的凭据，只有该应用程序才能访问这些凭据。这为应用程序提供了一种管理仅供其使用的凭据的方法，同时提供了与  `KeyChain` API 为系统级凭据提供的相同安全优势。此方法不需要用户选择凭据。此方法不需要用户选择凭据。

## 使用 Android 密钥库提供程序

要使用此功能，您需要使用标准的 [`KeyStore`](https://developer.android.com/reference/java/security/KeyStore) 和 [`KeyPairGenerator`](https://developer.android.com/reference/java/security/KeyPairGenerator) 或 [`KeyGenerator`](https://developer.android.com/reference/javax/crypto/KeyGenerator)，以及 Android 4.3（API 级别 18）中引入的 `AndroidKeyStore` 提供程序。

`AndroidKeyStore` 被注册为 `KeyStore` 类型，可用于 [`KeyStore.getInstance(type)`](https://developer.android.com/reference/java/security/KeyStore#getInstance\(java.lang.String\)) 方法，并被注册为提供程序，可用于 [`KeyPairGenerator.getInstance(algorithm, provider)`](https://developer.android.com/reference/java/security/KeyPairGenerator#getInstance\(java.lang.String,%20java.lang.String\)) 和 [`KeyGenerator.getInstance(algorithm, provider)`](https://developer.android.com/reference/javax/crypto/KeyGenerator#getInstance\(java.lang.String,%20java.lang.String\))方法。

由于加密操作可能耗时较长，应用程序应避免在主线程中使用  `AndroidKeyStore`，以确保应用程序的 UI 保持响应速度。（`StrictMode` 可以帮助您找到不符合此要求的地方。）

### 生成新私钥或密钥

要生成包含 [`PrivateKey`](https://developer.android.com/reference/java/security/PrivateKey)`的新`KeyPair，必须指定证书的初始 X.509 属性。您可以使用 [`KeyStore.setKeyEntry()`](https://developer.android.com/reference/java/security/KeyStore#setKeyEntry\(java.lang.String,%20java.security.Key,%20char[],%20java.security.cert.Certificate[]\)) 在稍后将证书替换为由证书颁发机构 (CA) 签名的证书。

要生成密钥对，请使用带有 [`KeyGenParameterSpec`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec) 的  [`KeyPairGenerator`](https://developer.android.com/reference/java/security/KeyPairGenerator)：

[Kotlin](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#kotlin)
```kotlin
/*
 * Generate a new EC key pair entry in the Android Keystore by
 * using the KeyPairGenerator API. The private key can only be
 * used for signing or verification and only with SHA-256 or
 * SHA-512 as the message digest.
 */
val kpg: KeyPairGenerator = KeyPairGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_EC,
        "AndroidKeyStore"
)
val parameterSpec: KeyGenParameterSpec = KeyGenParameterSpec.Builder(
        alias,
        KeyProperties.PURPOSE_SIGN or KeyProperties.PURPOSE_VERIFY
).run {
    setDigests(KeyProperties.DIGEST_SHA256, KeyProperties.DIGEST_SHA512)
    build()
}

kpg.initialize(parameterSpec)

val kp = kpg.generateKeyPair()
```
[Java](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#java)
```java
/*
 * Generate a new EC key pair entry in the Android Keystore by
 * using the KeyPairGenerator API. The private key can only be
 * used for signing or verification and only with SHA-256 or
 * SHA-512 as the message digest.
 */
KeyPairGenerator kpg = KeyPairGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_EC, "AndroidKeyStore");
kpg.initialize(new KeyGenParameterSpec.Builder(
        alias,
        KeyProperties.PURPOSE_SIGN | KeyProperties.PURPOSE_VERIFY)
        .setDigests(KeyProperties.DIGEST_SHA256,
            KeyProperties.DIGEST_SHA512)
        .build());

KeyPair kp = kpg.generateKeyPair();
```

### 将加密密钥导入安全硬件

Android 9（API 级别 28）及更高版本允许您使用 ASN.1 编码的密钥格式将加密密钥安全地导入密钥库。密钥管理程序随后对密钥库中的密钥进行解密，因此密钥内容永远不会以明文形式出现在设备的宿主机内存中。此过程提供了额外的密钥解密安全性。

**注意：** 只有搭载 Keymaster 4 或更高版本的设备才支持该功能。

如需支持以安全方式将已加密密钥导入密钥库，请完成以下步骤：

1. 生成一个使用  [`PURPOSE_WRAP_KEY`](https://developer.android.com/reference/android/security/keystore/KeyProperties?hl=zh-cn#PURPOSE_WRAP_KEY) 用途的密钥对。我们建议您也为该密钥对添加证明。
    
2. 在您信任的服务器或机器上，为 `SecureKeyWrapper` 生成 ASN.1 消息。
    
    该封装容器包含以下架构：
	```java
	KeyDescription ::= SEQUENCE {
           keyFormat INTEGER,
           authorizationList AuthorizationList
       }
    
       SecureKeyWrapper ::= SEQUENCE {
           wrapperFormatVersion INTEGER,
           encryptedTransportKey OCTET_STRING,
           initializationVector OCTET_STRING,
           keyDescription KeyDescription,
           secureKey OCTET_STRING,
           tag OCTET_STRING
       }
	```                        
    
    
3. 创建 [`WrappedKeyEntry`](https://developer.android.com/reference/android/security/keystore/WrappedKeyEntry?hl=zh-cn) 对象，传入字节数组形式的 ASN.1 消息。
    
4. 将此 `WrappedKeyEntry` 对象传递给接受 [`Keystore.Entry`](https://developer.android.com/reference/java/security/KeyStore.Entry?hl=zh-cn) 对象的 [`setEntry()`](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn#setEntry\(java.lang.String,%20java.security.KeyStore.Entry,%20java.security.KeyStore.ProtectionParameter\)) 的重载。
    

### 使用密钥库条目

您可以通过所有标准 [`KeyStore`](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn) API 访问 `AndroidKeyStore` 提供程序。

#### 列出条目

通过调用 [`aliases()`](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn#aliases\(\)) 方法列出密钥库中的条目：

[Kotlin](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#kotlin)
```kotlin
/*
 * Load the Android KeyStore instance using the
 * AndroidKeyStore provider to list the currently stored entries.
 */
val ks: KeyStore = KeyStore.getInstance("AndroidKeyStore").apply {
   load(null)
}
val aliases: Enumeration<String> = ks.aliases()
```

[Java](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#java)

```java
/*
 * Load the Android KeyStore instance using the
 * AndroidKeyStore provider to list the currently stored entries.
 */
KeyStore ks = KeyStore.getInstance("AndroidKeyStore");
ks.load(null);
Enumeration<String> aliases = ks.aliases();
```

#### 对数据进行签名和验证

通过从密钥库中获取 [`KeyStore.Entry`](https://developer.android.com/reference/java/security/KeyStore.Entry?hl=zh-cn) 并使用 [`Signature`](https://developer.android.com/reference/java/security/Signature?hl=zh-cn) API（例如 [`sign()`](https://developer.android.com/reference/java/security/Signature?hl=zh-cn#sign\(\))）来为数据签名：

[Kotlin](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#kotlin)
```kotlin
/*
 * Use a PrivateKey in the KeyStore to create a signature over
 * some data.
 */
val ks: KeyStore = KeyStore.getInstance("AndroidKeyStore").apply {
    load(null)
}
val entry: KeyStore.Entry = ks.getEntry(alias, null)
if (entry !is KeyStore.PrivateKeyEntry) {
    Log.w(TAG, "Not an instance of a PrivateKeyEntry")
    return null
}
val signature: ByteArray = Signature.getInstance("SHA256withECDSA").run {
    initSign(entry.privateKey)
    update(data)
    sign()
}
```

[Java](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#java)

```java
/*
 * Use a PrivateKey in the KeyStore to create a signature over
 * some data.
 */
KeyStore ks = KeyStore.getInstance("AndroidKeyStore");
ks.load(null);
KeyStore.Entry entry = ks.getEntry(alias, null);
if (!(entry instanceof PrivateKeyEntry)) {
    Log.w(TAG, "Not an instance of a PrivateKeyEntry");
    return null;
}
Signature s = Signature.getInstance("SHA256withECDSA");
s.initSign(((PrivateKeyEntry) entry).getPrivateKey());
s.update(data);
byte[] signature = s.sign();
```

类似地，请使用 [`verify(byte[])`](https://developer.android.com/reference/java/security/Signature?hl=zh-cn#verify\(byte%5B%5D\)) 方法验证数据：

[Kotlin](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#kotlin)
```kotlin
/*
 * Verify a signature previously made by a private key in the
 * KeyStore. This uses the X.509 certificate attached to the
 * private key in the KeyStore to validate a previously
 * generated signature.
 */
val ks = KeyStore.getInstance("AndroidKeyStore").apply {
    load(null)
}
val entry = ks.getEntry(alias, null) as? KeyStore.PrivateKeyEntry
if (entry == null) {
    Log.w(TAG, "Not an instance of a PrivateKeyEntry")
    return false
}
val valid: Boolean = Signature.getInstance("SHA256withECDSA").run {
    initVerify(entry.certificate)
    update(data)
    verify(signature)
}
```
[Java](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn#java)

```java
/*
 * Verify a signature previously made by a private key in the
 * KeyStore. This uses the X.509 certificate attached to the
 * private key in the KeyStore to validate a previously
 * generated signature.
 */
KeyStore ks = KeyStore.getInstance("AndroidKeyStore");
ks.load(null);
KeyStore.Entry entry = ks.getEntry(alias, null);
if (!(entry instanceof PrivateKeyEntry)) {
    Log.w(TAG, "Not an instance of a PrivateKeyEntry");
    return false;
}
Signature s = Signature.getInstance("SHA256withECDSA");
s.initVerify(((PrivateKeyEntry) entry).getCertificate());
s.update(data);
boolean valid = s.verify(signature);
```

### 要求进行用户身份验证才能使用密钥

生成密钥或将密钥导入到 `AndroidKeyStore` 时，您可以指定只有在用户通过身份验证后才能使用该密钥。用户身份验证使用其安全锁屏凭据（图案/PIN码/密码、生物识别凭据）的子集。

这是一项高级安全功能，通常仅在以下情况下才有用：即使您的应用程序流程在密钥生成/导入之后（而不是之前或期间）遭到破坏，也无法绕过用户使用密钥时需要进行身份验证的要求。

如果密钥仅在用户通过身份验证后才能使用，则可以调用 [`setUserAuthenticationParameters()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn#setUserAuthenticationParameters\(int,%20int\)) 将其配置为在以下模式之一中运行：

 - **授权在一段时间内有效** 在用户使用指定的凭据之一进行身份验证后，即授权其使用所有密钥。
 - **在特定加密操作期间进行授权** 涉及特定密钥的每个操作都必须由用户单独授权。应用可以通过对 `BiometricPrompt` 的实例调用 [`authenticate()`](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt?hl=zh-cn#authenticate\(android.hardware.biometrics.BiometricPrompt.CryptoObject,%20android.os.CancellationSignal,%20java.util.concurrent.Executor,%20android.hardware.biometrics.BiometricPrompt.AuthenticationCallback\)) 来启动此流程。

创建每个密钥时，您可以选择支持[安全系数高的生物识别凭据](https://developer.android.com/reference/android/security/keystore/KeyProperties?hl=zh-cn#AUTH_BIOMETRIC_STRONG)、[锁屏凭据](https://developer.android.com/reference/android/security/keystore/KeyProperties?hl=zh-cn#AUTH_DEVICE_CREDENTIAL)，或者两种凭据皆可支持。如需确定用户是否设置了应用密钥所依赖的凭据，请调用 [`canAuthenticate()`](https://developer.android.com/reference/android/hardware/biometrics/BiometricManager?hl=zh-cn#canAuthenticate\(int\))。

如果密钥仅支持生物识别凭据，则每当添加新的已注册生物识别信息时，该密钥都会默认失效。您可以配置密钥，使其在添加新的生物识别注册时仍然有效。为此，请将 false 传递给[`setInvalidatedByBiometricEnrollment()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn#setInvalidatedByBiometricEnrollment\(boolean\))。

详细了解如何在应用中添加生物识别验证功能，包括如何[显示生物识别验证对话框](https://developer.android.com/training/sign-in/biometric-auth?hl=zh-cn)。

## 支持的算法

- [`Cipher`](https://developer.android.com/reference/javax/crypto/Cipher?hl=zh-cn)
- [`KeyGenerator`](https://developer.android.com/reference/javax/crypto/KeyGenerator?hl=zh-cn)
- [`KeyFactory`](https://developer.android.com/reference/java/security/KeyFactory?hl=zh-cn)
- `KeyStore`（支持的密钥类型与 `KeyGenerator` 和 `KeyPairGenerator` 相同）
- [`KeyPairGenerator`](https://developer.android.com/reference/java/security/KeyPairGenerator?hl=zh-cn)
- [`Mac`](https://developer.android.com/reference/javax/crypto/Mac?hl=zh-cn)
- [`Signature`](https://developer.android.com/reference/java/security/Signature?hl=zh-cn)
- [`SecretKeyFactory`](https://developer.android.com/reference/javax/crypto/SecretKeyFactory?hl=zh-cn)

## 博客文章

请查看 Android 开发者博客上的文章 [在 ICS 中统一密钥库访问](http://android-developers.blogspot.com/2012/03/unifying-key-store-access-in-ics.html)。

