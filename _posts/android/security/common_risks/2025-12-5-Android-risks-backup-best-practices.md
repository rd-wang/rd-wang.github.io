---
title: Android-安全性-安全风险-备份安全建议
date: 2025-12-5 16:09:59 +0800
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
description: 遵循安全建议设置应用程序备份可以防止备份中可能包含的敏感数据泄露。根据实际数据和攻击者的意图，敏感数据泄露可能导致信息泄露、用户身份冒用和经济损失。
math: true
---
# 备份的安全建议

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

应用备份旨在保留用户的数据，以便日后在新设备或数据丢失时进行恢复。现有的有关应用备份的安全建议因 Android 版本和设备制造商而异。这些建议的共同主题是，旨在确保不会泄露任何敏感数据。

标准 Android 备份系统提供了最安全、最可靠且最简单的解决方案，可将数据备份到云端或通过[自动备份](https://developer.android.com/identity/data/autobackup)（[默认启用](https://developer.android.com/identity/data/backup)，无需任何额外设置，且可扩展）和[键值备份](https://developer.android.com/guide/topics/data/keyvaluebackup)将数据传输到新设备。我们推荐使用此方案，因为它会将备份数据存储在其他第三方应用无法访问的目录中，并支持静态加密、传输中加密，以及允许配置从备份中排除敏感数据的功能。

如果应用程序采用不依赖于标准 Android 备份系统的备份方案，则可能增加因操作失误导致敏感数据泄露的风险。例如，某些应用程序提供的“导出”或“备份”功能会将应用程序数据副本创建在其他应用程序可读取的目录中，因此这些副本很容易被泄露（无论是直接泄露还是通过其他漏洞泄露）。

## 影响

遵循安全建议设置应用程序备份可以防止备份中可能包含的敏感数据泄露。根据实际数据和攻击者的意图，敏感数据泄露可能导致信息泄露、用户身份冒用和经济损失。

## 缓解措施

### 使用标准 Android 备份系统

标准 Android 备份系统始终会对传输中的数据和静态数据进行加密。无论您使用的是哪个 Android 版本，无论您的设备是否有锁屏，系统都会应用此加密。从 Android 9 开始，如果设备设置了锁定屏幕，则备份数据不仅会被加密，还会使用 Google 未知的密钥进行加密（锁屏密码会保护加密密钥，从而实现端到端加密）。

一般来说，请务必遵循[数据存储](https://developer.android.com/training/data-storage?hl=zh-cn)和[安全准则](https://developer.android.com/privacy-and-security/risks/sensitive-data-external-storage?hl=zh-cn)。

如果您的备份包含特别敏感的数据，我们建议您排除此类数据；如果无法排除，则要求使用端到端加密（如以下部分所述）。

#### 从备份中排除数据

您可以使用规则文件（通常称为 `backup_rules.xml` 并放置在 `res/xml` 应用文件夹中）指定要从备份中排除哪些数据。根据所使用的 Android 版本，备份规则的配置方式略有不同：

- [对于 Android 12（API 级别 31）及更高版本](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-12)，请向 `AndroidManifest.xml` 中的 `<application>` 元素添加 `android:dataExtractionRules` 属性：
```xml 
<application android:name="com.example.foo" android:dataExtractionRules="@xml/backup_rules_extraction"> … </application>
```

然后，根据应用的数据持久性和安全性要求，按照[更新后的配置格式](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#xml-syntax-android-12)[配置](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-12) `backup_rules.xml` 文件。

`backup_rules.xml` 文件配置所需的格式允许开发者为云端和[设备到设备 (D2D) 传输](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn#xml-changes)定义自定义备份规则。如果未设置 `<device-transfer>` 属性，系统会在点对点迁移期间传输所有应用数据。请务必注意，即使目标应用以 Android 12 或更高版本为目标平台，也应始终为搭载 Android 11（API 级别 30）或更低版本的设备指定包含[一组额外备份规则](https://developer.android.com/identity/data/autobackup?hl=zh-cn#include-exclude-android-11)的单独文件。

- [对于 Android 11 及更低版本](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-11)，请将 `android:fullBackupContent` 属性添加到 `AndroidManifest.xml` 中的 `<application>` 元素：
```xml 
<application android:name="com.example.foo" android:fullBackupContent="@xml/backup_rules_full"> … </application>
```

然后，根据应用的数据持久性和安全要求，使用[备份用户数据](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-11)一文中报告的语法配置 `backup_rules.xml` 文件。

#### 要求使用端到端加密

如果您无法从备份中排除敏感数据，我们建议您要求进行端到端加密，这意味着仅允许在 Android 9 或更高版本上进行备份，并且仅在启用锁屏时才允许备份。您可以使用 `requireFlags="clientSideEncryption"` 标志来实现此目的，该标志需要从 [Android 12](https://developer.android.com/identity/data/autobackup?hl=zh-cn#include-exclude-android-12) 开始重命名为 `disableIfNoEncryptionCapabilities` 并设置为 `true`。

### 如果您无法使用标准 Android 备份系统

如果无法使用标准的 Android 备份系统，那么安全地存储备份数据以及指定要从备份中排除的数据就变得更加复杂。这需要在代码层面进行定义，因此容易出错，并存在数据泄露的风险。在这种情况下，建议定期测试您的实现，以确保备份行为符合预期。

## 资源

- [allowBackup 属性说明](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#allowbackup)
- [文件级加密](https://source.android.com/docs/security/features/encryption/file-based?hl=zh-cn)
- [D2D 传输行为变更](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn#functionality-changes)
- [通过自动备份功能备份用户数据](https://developer.android.com/identity/data/autobackup?hl=zh-cn)
- [使用 Android Backup Service 备份键值对](https://developer.android.com/identity/data/keyvaluebackup?hl=zh-cn)
- [在 Android 12 或更高版本上控制备份](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-12)
- [在 Android 11 及更低版本中控制备份](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn#include-exclude-android-11)
- [了解 Google 合同和政策中的个人身份信息](https://support.google.com/analytics/answer/7686480?hl=zh-cn)
- [测试备份和恢复功能](https://developer.android.com/identity/data/testingbackup?hl=zh-cn)
- [加密](https://developer.android.com/guide/topics/security/cryptography?hl=zh-cn)
- [Android 密钥库系统](https://developer.android.com/training/articles/keystore?hl=zh-cn)
- [ADB](https://developer.android.com/tools/adb?hl=zh-cn)
- [开发者选项](https://developer.android.com/studio/debug/dev-options?hl=zh-cn)

