---
title: Android-安全风险-路径遍历
date: 2025-12-5 15:00:39 +0800
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
description: 通过在目标目录之外进行遍历，攻击者可能会使用 ../ 等特殊字符以意想不到的方式更改资源目标
math: true
---


# 路径遍历

**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

当攻击者能够控制路径的一部分，因而导致该部分路径会在未经验证的情况下被传递到文件系统 API 时，则存在路径遍历漏洞。这可能会导致攻击者能够在未经授权的情况下操作文件系统。例如，通过在目标目录之外进行遍历，攻击者可能会使用 `../` 等特殊字符以意想不到的方式更改资源目标。

## 影响

具体影响因操作和文件内容而异，但通常会导致文件覆盖（写入文件时）、数据泄露（读取文件时）或权限更改（更改文件/目录权限时）。

## 缓解措施

使用 [`File.getCanonicalPath()`](https://developer.android.com/reference/java/io/File?hl=zh-cn#getCanonicalPath\(\)) 对路径进行规范化，并将前缀与预期目录进行比较：

[Kotlin](https://developer.android.com/privacy-and-security/risks/path-traversal?hl=zh-cn#kotlin)
```kotlin
@Throws(IllegalArgumentException::class)
fun saferOpenFile(path: String, expectedDir: String?): File {
    val f = File(path)
    val canonicalPath = f.canonicalPath
    require(canonicalPath.startsWith(expectedDir!!))
    return f
}
```
[Java](https://developer.android.com/privacy-and-security/risks/path-traversal?hl=zh-cn#java)

```java
public File saferOpenFile (String path, String expectedDir) throws IllegalArgumentException {
  File f = new File(path);
  String canonicalPath = f.getCanonicalPath();
  if (!canonicalPath.startsWith(expectedDir)) {
    throw new IllegalArgumentException();
  }
  return f;
}
```

另一种最佳实践是通过验证来确保仅发生预期结果。示例如下：

- 检查文件是否已经存在，以防发生意外覆盖。
- 检查目标文件是否为预期目标，以防止数据泄露或错误地更改权限。
- 检查相应操作的当前目录是否如预期一样，与来自规范路径的返回值中的目录完全一致。
- 确保权限系统的作用域明确限定为相应操作，例如确保其未将相关服务作为 root 来运行，并确保目录权限的作用域限定为指定的服务或命令。