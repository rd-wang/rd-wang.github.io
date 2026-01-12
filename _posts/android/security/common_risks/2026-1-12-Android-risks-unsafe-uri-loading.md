---
title: Android-安全性-安全风险-不安全的 URI 加载
date: 2026-1-12 11:20:55 +0800
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
description: 在 WebView 中加载恶意 URI（即绕过过滤/许可名单的 URI）的情况下，可能会导致账号被盗用（例如使用钓鱼式攻击）、代码执行（例如加载恶意 JavaScript）或设备被破解（利用通过超链接提供的代码）。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

当 Android 应用无法在 URI 加载到 WebView 中之前正确评估其有效性时，便会发生不安全的 URI 加载。

此类漏洞的根本原因在于 [URI 由多个部分组成](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/net/URI.html)，其中至少 scheme 和（授权方部分的）主机是必须经验证（例如加入许可名单）的，然后 URI 才能加载到 WebView 或由应用在内部使用。

最常见的错误包括：

- 检查主机但未检查 scheme，让攻击者可通过经身份验证的主机使用 `http://`、`content://` 或 `javascript://` 等 scheme。
- 未能正确解析 URI，尤其是在以字符串形式接收 URI 的情况中。
- 验证 scheme，但未验证主机（主机验证不充分）。

关于最后一种情况，它通常会在应用需要允许主域名的任意子网域时发生。因此，即使已正确提取主机名，应用也会使用 `java.lang.String` 类的 `startsWith`、`endsWith,` 或 `contains` 等方法来验证在所提取的字符串部分中是否存在主域名。如果使用不当，这些方法可能会导致错误结果，并迫使应用错误地信任可能存在恶意的主机。

## 影响

具体影响可能会因主机的使用环境而异。在 WebView 中加载恶意 URI（即绕过过滤/许可名单的 URI）的情况下，可能会导致账号被盗用（例如使用钓鱼式攻击）、代码执行（例如加载恶意 JavaScript）或设备被破解（利用通过超链接提供的代码）。

## 缓解措施

处理字符串 URI 时，请务必将字符串解析为 URI 并验证 scheme 和主机：

[Kotlin](https://developer.android.com/privacy-and-security/risks/unsafe-uri-loading?hl=zh-cn#kotlin)
```kotlin
fun isUriTrusted(incomingUri: String, trustedHostName: String): Boolean {
    try {
        val uri = Uri.parse(incomingUri)
        return uri.scheme == "https" && uri.host == trustedHostName
    } catch (e: NullPointerException) {
        throw NullPointerException("incomingUri is null or not well-formed")
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/unsafe-uri-loading?hl=zh-cn#java)

```java
public static boolean isUriTrusted(String incomingUri, String trustedHostName)
    throws NullPointerException {
        try {
            Uri uri = Uri.parse(incomingUri);
            return uri.getScheme().equals("https") &&
            uri.getHost().equals(trustedHostName);
        } catch (NullPointerException e) {
            throw new NullPointerException(
                "incomingUri is null or not well-formed");
        }
    }
```

对于主机验证，在隔离相应的 URI 部分后，请务必对其进行完整验证（而不是部分验证），以准确确定主机是否可信。当无法避免使用 `startsWith` 或 `endsWith` 等方法时，请务必使用正确的语法，并且不要忽略必要的字符或符号（例如，`endsWith` 需要在域名之前有“`.`”点字符才能准确匹配）。忽略这些字符可能导致匹配不准确，并降低安全性。由于子网域可以无限嵌套，因此我们不建议使用正则表达式匹配来验证主机名。

## 资源

- [`getHost()` 文档](https://developer.android.com/reference/java/net/URI?hl=zh-cn#getHost\(\))
- [`getScheme()` 文档](https://developer.android.com/reference/java/net/URI?hl=zh-cn#getScheme\(\))

