---
title: Android-安全性-安全风险-存储在外部存储空间中的敏感数据
date: 2025-12-23 10:58:04 +0800
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
description: 如果敏感数据存储在外部存储空间中，设备上具有 READ_EXTERNAL_STORAGE 权限的任何应用都可以访问这些数据。这会允许恶意应用静默访问永久或临时存储在外部存储空间中的敏感文件。
math: true
---
**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

以 Android 10（API 29）或更低版本为目标平台的应用不会强制执行[分区存储](https://developer.android.com/training/data-storage?hl=zh-cn#scoped-storage)。这意味着任何存储在外部存储上的数据都可以被任何其他拥有 READ_EXTERNAL_STORAGE [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE) 权限的应用程序访问。

## 影响

在面向 Android 10 (API 29) 或更低版本的应用程序中，如果敏感数据存储在外部存储中，则设备上任何具有 READ_EXTERNAL_STORAGE 权限的应用程序都可以访问它。这使得恶意应用程序能够静默地访问永久或临时存储在外部存储设备上的敏感文件。此外，由于系统上的任何应用程序都可以访问外部存储设备上的内容，任何声明了 WRITE_EXTERNAL_STORAGE 权限的恶意应用程序都可以篡改存储在外部存储上的文件，例如，添加恶意数据。如果将这些恶意数据加载到应用程序中，则可能被用来欺骗用户，甚至实现代码执行。

## 缓解措施

### 分区存储（Android 10 及更高版本）

##### Android 10

对于面向 Android 10 的应用，开发者可以显式地选择启用作用域存储。这可以通过在 AndroidManifest.xml 文件中将 [`requestLegacyExternalStorage`](https://developer.android.com/reference/android/R.attr?hl=zh-cn#requestLegacyExternalStorage)  标志设置为 false 来实现。
通过分区存储，应用程序只能访问其自身在外部存储上创建的文件，或者使用 [MediaStore API](https://developer.android.com/reference/android/provider/MediaStore?hl=zh-cn) 存储的文件类型，例如音频和视频。这有助于保护用户隐私和安全。

##### Android 11 及更高版本

对于面向 Android 11 或更高版本的应用程序，操作系统[会强制使用分区存储](https://developer.android.com/about/versions/11/privacy/storage?hl=zh-cn#scoped-storage)，即忽略 [`requestLegacyExternalStorage`](https://developer.android.com/reference/android/R.attr?hl=zh-cn#requestLegacyExternalStorage) 标志，并自动保护应用程序的外部存储免受未经授权的访问。

### 将内部存储空间用于敏感数据

无论目标 Android 版本如何，应用程序的敏感数据都应该始终存储在内部存储中。由于 Android 沙盒机制，对内部存储的访问会自动限制为只有其所属应用程序才能访问，因此可以认为它是安全的，除非设备已被 root。
### 加密敏感数据

如果应用程序的使用场景需要在外部存储设备上存储敏感数据，则应加密这些数据。建议使用强加密算法，并使用  [Android 密钥库](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)安全地存储密钥。

一般来说，建议对所有敏感数据进行加密，无论这些数据存储在何处。

请务必注意，全盘加密（或 Android 10 中的基于文件的加密）是一种旨在保护数据免受物理访问和其他攻击途径侵害的措施。因此，为了达到同样的安全效果，存储在外部存储设备上的敏感数据还应由应用程序进行加密。

### 执行完整性检查

在需要将数据或代码从外部存储加载到应用程序的情况下，建议进行完整性检查，以验证没有其他应用程序篡改这些数据或代码。文件哈希值应以安全的方式存储，最好加密后存储在内部存储器中。

[Kotlin](https://developer.android.com/privacy-and-security/risks/sensitive-data-external-storage?hl=zh-cn#kotlin)
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
        val sb = StringBuilder()
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

[Java](https://developer.android.com/privacy-and-security/risks/sensitive-data-external-storage?hl=zh-cn#java)

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
        StringBuilder sb = new StringBuilder();
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

## 资源

- [分区存储](https://developer.android.com/training/data-storage?hl=zh-cn#scoped-storage)
- [READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE)
- [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_EXTERNAL_STORAGE)
- [requestLegacyExternalStorage](https://developer.android.com/reference/android/R.attr?hl=zh-cn#requestLegacyExternalStorage)
- [数据和文件存储概览](https://developer.android.com/training/data-storage?hl=zh-cn)
- [数据存储（特定于应用）](https://developer.android.com/training/data-storage/app-specific?hl=zh-cn)
- [加密](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn)
- [密钥库](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)
- [文件级加密](https://source.android.com/docs/security/features/encryption/file-based?hl=zh-cn)
- [全盘加密](https://source.android.com/docs/security/features/encryption/full-disk?hl=zh-cn)

