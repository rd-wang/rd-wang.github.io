---
title: Android-安全性-安全风险-XML 外部实体注入 (XXE)
date: 2026-1-12 10:45:04 +0800
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
description: 当应用解析 XML 文档时，它可以处理文档中包含的任何 DTD（文档类型定义，也称为外部实体）。攻击者可以通过以 DTD 的形式注入恶意代码来利用此行为。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

XML 外部实体注入 (XXE) 是一种针对解析 XML 输入的应用的攻击。当包含对外部实体的引用的不可信 XML 输入由配置较弱的 XML 解析器处理时，就会发生 XXE 攻击。此攻击可用于预演多种突发事件，包括拒绝服务攻击、文件系统访问或数据渗漏。

## 影响

当应用解析 XML 文档时，它可以处理文档中包含的任何 DTD（文档类型定义，也称为外部实体）。攻击者可以通过以 DTD 的形式注入恶意代码来利用此行为。然后，该代码可以访问设备文件系统的部分内容，这些内容只有应用程序才能访问，并且可能包含敏感数据。此外，此恶意代码可以从设备发出请求，可能会绕过边界安全措施。

最后，如果应用展开 DTD，可能会导致引用的实体多次迭代，耗尽设备资源并导致拒绝服务。

## 缓解措施

### 禁用 DTD

防止 XXE 的最安全方法是始终完全禁用 DTD（外部实体）。根据所使用的解析器，该方法可能类似于以下示例中的 XML Pull Parser 库示例：

[Java](https://developer.android.com/privacy-and-security/risks/xml-external-entities-injection?hl=zh-cn#java)
```java
XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```
[Kotlin](https://developer.android.com/privacy-and-security/risks/xml-external-entities-injection?hl=zh-cn#kotlin)

```kotlin
val factory = XmlPullParserFactory.newInstance()
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)
```

禁用 DTD 还可以增强解析器抵御拒绝服务攻击的能力。如果无法完全禁用 DTD，则必须针对每个解析器以特定方式禁用外部实体和外部文档类型声明。

由于市场上有大量 XML 解析引擎，因此防范 XXE 攻击的方法因引擎而异。如需了解详情，您可能需要参阅引擎文档。

### 执行输入清理

您应重新配置应用，使其不允许用户在 XML 文档的前导中注入任意代码。这必须在服务器端进行验证，因为客户端控件可能会被绕过。

### 使用其他库

如果所用库或方法无法以安全的方式进行配置，则应考虑使用其他库或方法。[XML 拉取解析器](https://developer.android.com/reference/org/xmlpull/v1/XmlPullParser?hl=zh-cn)和 [SAX 解析器](https://developer.android.com/reference/javax/xml/parsers/SAXParser?hl=zh-cn)都可以以安全的方式进行配置，禁止使用 DTD 和实体。

## 资源

- [OWASP XXE](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_\(XXE\)_Processing)
- [OWASP XXE 防范备忘单](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
- [XML 常量：FEATURE_SECURE_PROCESSING](https://developer.android.com/reference/javax/xml/XMLConstants?hl=zh-cn#FEATURE_SECURE_PROCESSING)
- [XML 拉取解析器](https://developer.android.com/reference/org/xmlpull/v1/XmlPullParser?hl=zh-cn)
- [SAX 解析器](https://developer.android.com/reference/javax/xml/parsers/SAXParser?hl=zh-cn)

