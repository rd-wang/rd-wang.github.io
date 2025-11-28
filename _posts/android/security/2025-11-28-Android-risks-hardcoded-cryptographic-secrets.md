---
title: Android-安全风险-硬编码的加密密钥
date: 2025-11-28 15:52:20 +0800
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
description: 掌握逆向工程工具的攻击者可以非常轻松地检索硬编码的 Secret。但在很多情况下，此漏洞可能会导致重大的安全问题。
math: true
---

# 硬编码的加密密钥

**OWASP 类别：**[MASVS-CRYPTO：加密](https://mas.owasp.org/MASVS/06-MASVS-CRYPTO)

## 概览

开发者会利用加密技术，通过可靠的算法保护数据的机密性和完整性。然而密钥存储的使用偏少，并且通常通过代码或资源文件（如 `strings.xml`）以字符串或字节数组的形式硬编码到应用中。如果在应用的任何文件中公开 Secret，就会违反 [Kerchoff 的原则](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle)，并且安全模型可能会被视为已损坏。

## 影响

掌握逆向工程工具的攻击者可以非常轻松地检索硬编码的 Secret。具体影响可能会因情况而异，但在很多情况下，此漏洞可能会导致重大的安全问题，例如对敏感数据的不当访问。

## 缓解措施

可以通过以下方法缓解此问题：如果您希望采用系统级凭据，可考虑使用 [KeyChain](https://developer.android.com/reference/android/security/KeyChain?hl=zh-cn) API；如果您想让单个应用可以存储自己的凭据，并且只有该应用本身可以访问相应凭据，可使用 [Android 密钥库](https://developer.android.com/training/articles/keystore?hl=zh-cn)提供程序。

以下代码段展示了如何利用 `KeyStore` 存储和使用对称密钥：

[Kotlin](https://developer.android.com/privacy-and-security/risks/hardcoded-cryptographic-secrets?hl=zh-cn#kotlin)

```kotlin
private val ANDROID_KEY_STORE_PROVIDER = "AndroidKeyStore"
private val ANDROID_KEY_STORE_ALIAS = "AES_KEY_DEMO"

@Throws(
    KeyStoreException::class,
    NoSuchAlgorithmException::class,
    NoSuchProviderException::class,
    InvalidAlgorithmParameterException::class
)
private fun createAndStoreSecretKey() {
    val builder: KeyGenParameterSpec.Builder = KeyGenParameterSpec.Builder(
        ANDROID_KEY_STORE_ALIAS,
        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
    )
    val keySpec: KeyGenParameterSpec = builder
        .setKeySize(256)
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .setRandomizedEncryptionRequired(true)
        .build()
    val aesKeyGenerator: KeyGenerator =
        KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEY_STORE_PROVIDER)
    aesKeyGenerator.init(keySpec)
    val key: SecretKey = aesKeyGenerator.generateKey()
}

@Throws(
    KeyStoreException::class,
    UnrecoverableEntryException::class,
    NoSuchAlgorithmException::class,
    CertificateException::class,
    IOException::class,
    NoSuchPaddingException::class,
    InvalidKeyException::class,
    IllegalBlockSizeException::class,
    BadPaddingException::class
)
private fun encryptWithKeyStore(plainText: String): ByteArray? {
    // Initialize KeyStore
    val keyStore: KeyStore = KeyStore.getInstance(ANDROID_KEY_STORE_PROVIDER)
    keyStore.load(null)
    // Retrieve the key with alias androidKeyStoreAlias created before
    val keyEntry: KeyStore.SecretKeyEntry =
        keyStore.getEntry(ANDROID_KEY_STORE_ALIAS, null) as KeyStore.SecretKeyEntry
    val key: SecretKey = keyEntry.secretKey
    // Use the secret key at your convenience
    val cipher: Cipher = Cipher.getInstance("AES/GCM/NoPadding")
    cipher.init(Cipher.ENCRYPT_MODE, key)
    return cipher.doFinal(plainText.toByteArray())
}
```
[Java](https://developer.android.com/privacy-and-security/risks/hardcoded-cryptographic-secrets?hl=zh-cn#java)

```java
static private final String ANDROID_KEY_STORE_PROVIDER = "AndroidKeyStore";
static private final String ANDROID_KEY_STORE_ALIAS = "AES_KEY_DEMO";

private void createAndStoreSecretKey() throws KeyStoreException, NoSuchAlgorithmException, NoSuchProviderException, InvalidAlgorithmParameterException {
    KeyGenParameterSpec.Builder builder = new KeyGenParameterSpec.Builder(
        ANDROID_KEY_STORE_ALIAS,
        KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT);
    KeyGenParameterSpec keySpec = builder
        .setKeySize(256)
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .setRandomizedEncryptionRequired(true)
        .build();
    KeyGenerator aesKeyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEY_STORE_PROVIDER);
    aesKeyGenerator.init(keySpec);
    SecretKey key = aesKeyGenerator.generateKey();
}

private byte[] encryptWithKeyStore(final String plainText) throws KeyStoreException, UnrecoverableEntryException, NoSuchAlgorithmException, CertificateException, IOException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException {
    // Initialize KeyStore
    KeyStore keyStore = KeyStore.getInstance(ANDROID_KEY_STORE_PROVIDER);
    keyStore.load(null);
    // Retrieve the key with alias ANDROID_KEY_STORE_ALIAS created before
    KeyStore.SecretKeyEntry keyEntry = (KeyStore.SecretKeyEntry) keyStore.getEntry(ANDROID_KEY_STORE_ALIAS, null);
    SecretKey key = keyEntry.getSecretKey();
    // Use the secret key at your convenience
    final Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.ENCRYPT_MODE, key);
    return cipher.doFinal(plainText.getBytes());
}
```

## 资源

- [Android 密钥库系统](https://developer.android.com/training/articles/keystore?hl=zh-cn)
    
- [密钥库文档](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn)
    
- [KeyChain 文档](https://developer.android.com/reference/android/security/KeyChain?hl=zh-cn)
    
- [CWE-321：使用硬编码的加密密钥](https://cwe.mitre.org/data/definitions/321.html)

