---
title: Android-安全风险-配置不当的FileProvider
date: 2025-11-28 16:13:14 +0800
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
description: 配置不当的 FileProvider 可能会意外向攻击者泄露文件和目录。
math: true
---

# 向 FileProvider 不当披露目录

**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

配置不当的 `FileProvider` 可能会意外向攻击者泄露文件和目录。根据具体配置，攻击者可能会读取或写入这些已泄露的文件，进而导致敏感信息渗漏，在最糟糕的情况下，攻击者可能会执行任意代码。例如，如果在应用的配置中设置了 `<root-path>`，攻击者就能访问在数据库中存储的敏感信息，或覆盖应用的原生库，从而执行任意代码。

## 影响

具体影响因配置和文件内容而异，但通常会导致数据泄露（读取时）或文件覆盖（写入时）。

## 缓解措施

### 请勿在配置中使用 `<root-path>` 路径元素

`<root-path>` 对应于设备的根目录 (`/`)。如果在配置中使用此元素，他人便可以随意访问文件和文件夹，包括应用的沙盒和 `/sdcard` 目录，这会向攻击者提供非常广的攻击面。

### 共享狭窄的路径范围

在路径配置文件中，避免共享宽泛的路径范围，如 `.` 或 `/`，否则可能会导致敏感文件意外泄露。请仅共享有限/更窄的路径范围，并确保此路径下只有您要共享的文件。这样可以防止敏感文件意外泄露。

采用更安全设置的典型配置文件如下所示：

[Xml](https://developer.android.com/privacy-and-security/risks/file-providers?hl=zh-cn#xml)

```xml
<paths>
    <files-path name="images" path="images/" />
    <files-path name="docs" path="docs" />
    <cache-path name="cache" path="net-export/" />
</paths>
```

### 检查并验证外部 URI

请验证外部 URI（使用 `content` 架构），并确保它们未指向应用的本地文件。这样可以防止信息意外泄露。

### 授予最低访问权限

[`content URI`](https://developer.android.com/guide/topics/providers/content-provider-basics?hl=zh-cn#ContentURIs) 可以同时具有读取和写入访问权限。请确保只授予所需的最低访问权限。 例如，如果仅需要读取权限，则仅明确授予 [`FLAG_GRANT_READ_URI_PERMISSION`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#FLAG_GRANT_READ_URI_PERMISSION)。

### 避免使用 `<external-path>` 存储/共享敏感信息

个人身份信息 (PII) 等敏感数据不应存储在应用容器或系统凭据存储设施之外的地方。因此，除非您明确确认存储/共享的信息不是敏感信息，否则请避免使用 `<external-path>` 元素。

## 资源

- [FileProvider 文档](https://developer.android.com/reference/androidx/core/content/FileProvider?hl=zh-cn)
    
- [使用 `<root-path>` 的漏洞](https://hackerone.com/reports/876192)
