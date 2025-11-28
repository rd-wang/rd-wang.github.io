---
title: Android-安全风险-不信任ContentProvider提供的文件名
date: 2025-11-28 16:34:25 +0800
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
description: 如果攻击者可以覆盖应用的文件，这可能会导致恶意代码执行
math: true
---

# 不信任 ContentProvider 提供的文件名

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

[_FileProvider_](https://developer.android.com/reference/androidx/core/content/FileProvider?hl=zh-cn) 是 [_ContentProvider_](https://developer.android.com/reference/android/content/ContentProvider?hl=zh-cn) 的子类，旨在为应用程序（“服务器应用程序”）与另一个应用程序（“客户端应用程序”）[共享文件](https://developer.android.com/training/secure-file-sharing?hl=zh-cn)提供一种安全的方法。不过，如果客户端应用未正确处理服务器应用提供的文件名，攻击者控制的服务器应用或许能够实现自己的恶意 _FileProvider_，以覆盖客户端应用的应用专用存储空间中的文件。

## 影响

如果攻击者可以覆盖应用的文件，这可能会导致恶意代码执行（通过覆盖应用的代码），或者允许以其他方式修改应用的行为（例如，通过覆盖应用的共享偏好设置或其他配置文件）。

## 缓解措施

### 不信任用户输入

在使用文件系统调用时，最好避免用户输入，而是在将接收到的文件写入存储设备时生成唯一的文件名。

换句话说：当客户端应用程序将接收到的文件写入存储时，它应该忽略服务器应用程序提供的文件名，而使用其内部生成的唯一标识符作为文件名。

此示例基于以下网址中的代码构建：[https://developer.android.com/training/secure-file-sharing/request-file](https://developer.android.com/training/secure-file-sharing/request-file?hl=zh-cn#java)：

[Kotlin](https://developer.android.com/privacy-and-security/risks/untrustworthy-contentprovider-provided-filename?hl=zh-cn#kotlin)

```kotlin
// Code in
// https://developer.android.com/training/secure-file-sharing/request-file#OpenFile
// used to obtain file descriptor (fd)

try {
    val inputStream = FileInputStream(fd)
    val tempFile = File.createTempFile("temp", null, cacheDir)
    val outputStream = FileOutputStream(tempFile)
    val buf = ByteArray(1024)
    var len: Int
    len = inputStream.read(buf)
    while (len > 0) {
        if (len != -1) {
            outputStream.write(buf, 0, len)
            len = inputStream.read(buf)
        }
    }
    inputStream.close()
    outputStream.close()
} catch (e: IOException) {
    e.printStackTrace()
    Log.e("MainActivity", "File copy error.")
    return
}
```
[Java](https://developer.android.com/privacy-and-security/risks/untrustworthy-contentprovider-provided-filename?hl=zh-cn#java)

```java
// Code in
// https://developer.android.com/training/secure-file-sharing/request-file#OpenFile
// used to obtain file descriptor (fd)

FileInputStream inputStream = new FileInputStream(fd);

// Create a temporary file
File tempFile = File.createTempFile("temp", null, getCacheDir());

// Copy the contents of the file to the temporary file
try {
    OutputStream outputStream = new FileOutputStream(tempFile))
    byte[] buffer = new byte[1024];
    int length;
    while ((length = inputStream.read(buffer)) > 0) {
        outputStream.write(buffer, 0, length);
    }
} catch (IOException e) {
    e.printStackTrace();
    Log.e("MainActivity", "File copy error.");
    return;
}
```

### 清理提供的文件名

在将接收到的文件写入存储空间时，对提供的文件名进行清理。

此缓解措施不如上述缓解措施理想，因为它难以处理所有潜在情况。尽管如此：如果生成唯一文件名不切实际，客户端应用程序应处理提供的文件名。处理方法包括：

- 清理文件名中的路径遍历字符
- 执行规范化以确认不存在路径遍历字符

此示例代码基于有关[检索文件信息](https://developer.android.com/training/secure-file-sharing/retrieve-info?hl=zh-cn)的指南：

[Kotlin](https://developer.android.com/privacy-and-security/risks/untrustworthy-contentprovider-provided-filename?hl=zh-cn#kotlin)

```kotlin
protected fun sanitizeFilename(displayName: String): String {
    val badCharacters = arrayOf("..", "/")
    val segments = displayName.split("/")
    var fileName = segments[segments.size - 1]
    for (suspString in badCharacters) {
        fileName = fileName.replace(suspString, "_")
    }
    return fileName
}

val displayName = returnCursor.getString(nameIndex)
val fileName = sanitizeFilename(displayName)
val filePath = File(context.filesDir, fileName).path

// saferOpenFile defined in Android developer documentation
val outputFile = saferOpenFile(filePath, context.filesDir.canonicalPath)

// fd obtained using Requesting a shared file from Android developer
// documentation

val inputStream = FileInputStream(fd)

// Copy the contents of the file to the new file
try {
    val outputStream = FileOutputStream(outputFile)
    val buffer = ByteArray(1024)
    var length: Int
    while (inputStream.read(buffer).also { length = it } > 0) {
        outputStream.write(buffer, 0, length)
    }
} catch (e: IOException) {
    // Handle exception
}

```
[Java](https://developer.android.com/privacy-and-security/risks/untrustworthy-contentprovider-provided-filename?hl=zh-cn#java)
```java
protected String sanitizeFilename(String displayName) {
    String[] badCharacters = new String[] { "..", "/" };
    String[] segments = displayName.split("/");
    String fileName = segments[segments.length - 1];
    for (String suspString : badCharacters) {
        fileName = fileName.replace(suspString, "_");
    }
    return fileName;
}

String displayName = returnCursor.getString(nameIndex);
String fileName = sanitizeFilename(displayName);
String filePath = new File(context.getFilesDir(), fileName).getPath();

// saferOpenFile defined in Android developer documentation

File outputFile = saferOpenFile(filePath,
    context.getFilesDir().getCanonicalPath());

// fd obtained using Requesting a shared file from Android developer
// documentation

FileInputStream inputStream = new FileInputStream(fd);

// Copy the contents of the file to the new file
try {
    OutputStream outputStream = new FileOutputStream(outputFile))
    byte[] buffer = new byte[1024];
    int length;
    while ((length = inputStream.read(buffer)) > 0) {
        outputStream.write(buffer, 0, length);
    }
} catch (IOException e) {
    // Handle exception
}
```
贡献者：Microsoft 威胁情报的 Dimitrios Valsamaras 和 Michael Peck

## 资源

- [Dirty Stream 攻击：将 Android Share Target 变成攻击途径](https://i.blackhat.com/Asia-23/AS-23-Valsamaras-Dirty-Stream-Attack-Turning-Android.pdf)
- [安全的文件共享](https://developer.android.com/training/secure-file-sharing?hl=zh-cn)
- [“请求某个分享的文件”文档](https://developer.android.com/training/secure-file-sharing/request-file?hl=zh-cn)
- [检索信息](https://developer.android.com/training/secure-file-sharing/retrieve-info?hl=zh-cn)
- [FileProvider](https://developer.android.com/reference/androidx/core/content/FileProvider?hl=zh-cn)
- [路径遍历](https://developer.android.com/topic/security/risks/path-traversal?hl=zh-cn)
- [CWE-73 文件名或路径的外部控制](https://cwe.mitre.org/data/definitions/73)