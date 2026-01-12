---
title: Android-安全性-安全风险-压缩路径遍历
date: 2026-1-12 11:30:39 +0800
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
description: 压缩路径遍历漏洞可被利用来覆盖任意文件。具体影响可能会因情况而异，但在很多情况下，此漏洞可能会导致代码执行等重大安全问题。
math: true
---
**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

“压缩路径遍历”漏洞（也称为 ZipSlip）与处理压缩归档文件有关。在本页中，我们将以 ZIP 格式为例介绍此漏洞，但处理其他格式（例如 TAR、RAR 或 7z）的库中也可能出现类似问题。

此问题的根本原因在于 ZIP 归档文件内存储的每个压缩文件都采用完全限定名称，此类名称允许使用斜杠和点等特殊字符。`java.util.zip` 软件包中的默认库不会检查归档条目的名称中是否包含路径遍历字符 (`../`)，因此在将从归档文件中提取的名称与目标目录路径串联时必须格外小心。

请务必对来自外部来源且提取 ZIP 的代码段或库进行验证。**许多此类库都容易受到压缩路径遍历攻击。**

## 影响

压缩路径遍历漏洞可被利用来覆盖任意文件。具体影响可能会因情况而异，但在很多情况下，此漏洞可能会导致代码执行等重大安全问题。

## 缓解措施

为缓解此问题，在提取每个条目之前，您应始终验证目标路径是否为目标目录的子级。以下代码假定目标目录是安全的（只能由您的应用写入，不受攻击者控制），否则您的应用程序可能容易受到其他漏洞的影响，例如符号链接攻击。

[Kotlin](https://developer.android.com/privacy-and-security/risks/zip-path-traversal?hl=zh-cn#kotlin)
```kotlin
companion object {
    @Throws(IOException::class)
    fun newFile(targetPath: File, zipEntry: ZipEntry): File {
        val name: String = zipEntry.name
        val f = File(targetPath, name)
        val canonicalPath = f.canonicalPath
        if (!canonicalPath.startsWith(
                targetPath.canonicalPath + File.separator)) {
            throw ZipException("Illegal name: $name")
        }
        return f
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/zip-path-traversal?hl=zh-cn#java)

```java
public static File newFile(File targetPath, ZipEntry zipEntry) throws IOException {
    String name = zipEntry.getName();
    File f = new File(targetPath, name);
    String canonicalPath = f.getCanonicalPath();
    if (!canonicalPath.startsWith(targetPath.getCanonicalPath() + File.separator)) {
      throw new ZipException("Illegal name: " + name);
    }
    return f;
 }
```

为避免意外覆盖现有文件，您还应确保目标目录在提取过程开始前为空。否则，可能会导致应用程序崩溃，在极端情况下，甚至可能导致应用程序被入侵。

[Kotlin](https://developer.android.com/privacy-and-security/risks/zip-path-traversal?hl=zh-cn#kotlin)
```kotlin
@Throws(IOException::class)
fun unzip(inputStream: InputStream?, destinationDir: File) {
    if (!destinationDir.isDirectory) {
        throw IOException("Destination is not a directory.")
    }
    val files = destinationDir.list()
    if (files != null && files.isNotEmpty()) {
        throw IOException("Destination directory is not empty.")
    }
    ZipInputStream(inputStream).use { zipInputStream ->
        var zipEntry: ZipEntry
        while (zipInputStream.nextEntry.also { zipEntry = it } != null) {
            val targetFile = File(destinationDir, zipEntry.name)
            // ...
        }
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/zip-path-traversal?hl=zh-cn#java)

```java
void unzip(final InputStream inputStream, File destinationDir)
      throws IOException {
  if(!destinationDir.isDirectory()) {
    throw IOException("Destination is not a directory.");
  }

  String[] files = destinationDir.list();
  if(files != null && files.length != 0) {
    throw IOException("Destination directory is not empty.");
  }

  try (ZipInputStream zipInputStream = new ZipInputStream(inputStream)) {
    ZipEntry zipEntry;
    while ((zipEntry = zipInputStream.getNextEntry()) != null) {
      final File targetFile = new File(destinationDir, zipEntry);
        …
    }
  }
}
```

## 资源

- [Zip Slip 漏洞](https://snyk.io/research/zip-slip-vulnerability)

