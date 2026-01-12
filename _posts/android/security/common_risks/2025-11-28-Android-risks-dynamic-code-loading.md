---
title: Android-安全性-安全风险-动态代码加载
date: 2025-11-28 15:39:47 +0800
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
description: 如果攻击者设法获得对要加载到应用中的代码的访问权限，则可以修改该代码以实现其目标。这可能会导致数据渗漏和代码执行漏洞。除非有业务需求，否则请避免动态代码加载。
math: true
---
# 动态代码加载

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

将代码动态加载到应用程序中会引入一定的风险，必须采取措施降低这种风险。攻击者可能会篡改或替换代码，以访问敏感数据或执行有害操作。

许多形式的动态代码加载（尤其是使用远程来源的加载方式）[违反了 Google Play 政策](https://support.google.com/googleplay/android-developer/answer/9888379?hl=zh-cn)，可能会导致应用在 Google Play 中被暂停。

## 影响

如果攻击者设法获取了将要加载到应用程序中的代码，他们就可以修改代码以实现其目标。这可能导致数据泄露和代码执行漏洞利用。即使攻击者无法修改代码来执行任意操作，他们仍然有可能破坏或删除代码，从而影响应用程序的可用性。

## 缓解措施

### 避免使用动态代码加载

除非有业务需要，否则应避免动态加载代码。应尽可能将所有功能直接集成到应用程序中。

### 使用可信来源

加载到应用程序中的代码应存储在可信位置。对于本地存储，建议使用应用程序内部存储或作用域存储（适用于 Android 10 及更高版本）。这些位置采取了措施，以避免其他应用程序和用户直接访问。

从网址等远程位置加载代码时，请尽可能避免使用第三方代码，并遵循安全最佳实践将代码存储在您自己的基础架构中。如果您需要加载第三方代码，请确保提供商是可信的。

### 执行完整性检查

建议进行完整性检查，以确保代码未遭到篡改。应在将代码加载到应用之前执行这些检查。

加载远程资源时，可以使用子资源完整性来验证所访问资源的完整性。

从外部存储空间加载资源时，请使用完整性检查来验证没有其他应用篡改过此数据或代码。文件的哈希值应以安全的方式存储，最好是加密并存储在内部存储空间中。

[Kotlin](https://developer.android.com/privacy-and-security/risks/dynamic-code-loading#kotlin)
```kotlin
package com.example.myapplication

import java.io.BufferedInputStream
import java.io.FileInputStream
import java.io.IOException
import java.security.MessageDigest
import java.security.NoSuchAlgorithmException

object FileIntegrityChecker {
    @Throws(IOException::class, NoSuchAlgorithmException::class)
    fun getIntegrityHash(filePath: String?): String {
        val md = MessageDigest.getInstance("SHA-256") // You can choose other algorithms as needed
        val buffer = ByteArray(8192)
        var bytesRead: Int
        BufferedInputStream(FileInputStream(filePath)).use { fis ->
            while (fis.read(buffer).also { bytesRead = it } != -1) {
                md.update(buffer, 0, bytesRead)
            }

    }

    private fun bytesToHex(bytes: ByteArray): String {
        val sb = StringBuilder(bytes.length * 2)
        for (b in bytes) {
            sb.append(String.format("%02x", b))
        }
        return sb.toString()
    }

    @Throws(IOException::class, NoSuchAlgorithmException::class)
    fun verifyIntegrity(filePath: String?, expectedHash: String): Boolean {
        val actualHash = getIntegrityHash(filePath)
        return actualHash == expectedHash
    }

    @Throws(Exception::class)
    @JvmStatic
    fun main(args: Array<String>) {
        val filePath = "/path/to/your/file"
        val expectedHash = "your_expected_hash_value"
        if (verifyIntegrity(filePath, expectedHash)) {
            println("File integrity is valid!")
        } else {
            println("File integrity is compromised!")
        }
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/dynamic-code-loading#java)
```java
package com.example.myapplication;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class FileIntegrityChecker {

    public static String getIntegrityHash(String filePath) throws IOException, NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("SHA-256"); // You can choose other algorithms as needed
        byte[] buffer = new byte[8192];
        int bytesRead;

        try (BufferedInputStream fis = new BufferedInputStream(new FileInputStream(filePath))) {
            while ((bytesRead = fis.read(buffer)) != -1) {
                md.update(buffer, 0, bytesRead);
            }
        }

        byte[] digest = md.digest();
        return bytesToHex(digest);
    }

    private static String bytesToHex(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 2);
        for (byte b : bytes) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }

    public static boolean verifyIntegrity(String filePath, String expectedHash) throws IOException, NoSuchAlgorithmException {
        String actualHash = getIntegrityHash(filePath);
        return actualHash.equals(expectedHash);
    }

    public static void main(String[] args) throws Exception {
        String filePath = "/path/to/your/file";
        String expectedHash = "your_expected_hash_value";

        if (verifyIntegrity(filePath, expectedHash)) {
            System.out.println("File integrity is valid!");
        } else {
            System.out.println("File integrity is compromised!");
        }
    }
}
```

### 对代码进行签名

确保数据完整性的另一种方法是在加载代码之前对其进行签名并验证其签名。这种方法的优势在于，不仅能确保代码本身的完整性，还能确保哈希代码的完整性，从而提供额外的防篡改保护。

虽然代码签名可提供额外的安全层，但请务必注意，这是一个更复杂的过程，可能需要额外的努力和资源才能成功实现。

您可以在本文档的“资源”部分找到一些代码签名示例。

## 资源

- [子资源完整性](https://en.wikipedia.org/wiki/Subresource_Integrity)
- [对数据进行数字签名](https://developers.google.com/tink/digitally-sign-data?hl=zh-cn#java)
- [代码签名](https://en.wikipedia.org/wiki/Code_signing)
- [存储在外部存储空间中的敏感数据](https://developer.android.com/privacy-and-security/risks/sensitive-data-external-storage?hl=zh-cn)
