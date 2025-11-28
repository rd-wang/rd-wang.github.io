---
title: Android-安全风险-自定义权限
date: 2025-11-28 12:04:52 +0800
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
description: 攻击者可能会通过创建具有相同名称、但由恶意应用定义且应用了不同保护级别的自定义权限来利用自定义权限相关的风险。
math: true
---

# 自定义权限

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

当自定义权限定义缺失或拼写错误，或者清单中的相应 `android:protectionLevel` 属性遭到滥用时，就会出现与自定义权限相关的风险。

例如，攻击者可能会通过创建具有相同名称、但由恶意应用定义且应用了不同保护级别的自定义权限来利用这些风险。

自定义权限旨在让应用能够与其他应用共享资源和功能。以下是合法使用自定义权限的示例：

- 控制两个或更多应用之间的进程间通信 (IPC)
- 访问第三方服务
- 限制对应用共享数据的访问权限

## 影响

利用此漏洞的影响是，恶意应用可能会获得对原本应受保护的资源的访问权限。此漏洞的影响取决于受保护的资源以及原始应用服务的关联权限。

## 风险：自定义权限拼写错误

清单中可能会声明自定义权限，但由于拼写错误，系统使用了其他自定义权限来保护导出的 Android 组件。恶意应用可能会利用通过以下任一方式拼错权限的应用：

- 先注册该权限
- 在后续应用中预测拼写

这可能会导致应用未经授权访问资源或控制受害应用。

例如，一个易受攻击的应用想要使用权限 `READ_CONTACTS` 保护某个组件，但不小心将该权限拼写错误为 `READ_CONACTS`。由于 `READ_CONACTS` 不归任何应用（或系统）所有，因此恶意应用可以声明对 `READ_CONACTS` 的所有权，并获得对受保护组件的访问权限。此漏洞的另一个常见变体是 `android:permission=True`。`true` 和 `false` 等值（无论大小写如何）都是权限声明的无效输入，并且会被视为其他自定义权限声明拼写错误。如需解决此问题，应将 `android:permission` 属性的值更改为有效的权限字符串。例如，如果应用需要访问用户的通讯录，则 `android:permission` 属性的值应为 `android.permission.READ_CONTACTS`。

### 缓解措施

#### Android Lint 检查

声明自定义权限时，请使用 Android lint 检查来帮助您查找代码中的拼写错误和其他潜在错误。

#### 命名惯例

使用一致的命名惯例，以便更容易发现拼写错误。仔细检查应用清单中的自定义权限声明，确保没有拼写错误。

---

## 风险：孤立权限

权限用于保护应用的资源。应用可在两个不同的位置声明访问资源所需的权限：

- AndroidManifest.xml：在 AndroidManifest.xml 文件中进行了预定义（如果未指定，将使用 `<application>` 权限），例如[provider 权限](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn#prmsn)、[receiver 权限](https://developer.android.com/guide/topics/manifest/receiver-element?hl=zh-cn#prmsn)、[activity 权限](https://developer.android.com/guide/topics/manifest/activity-element?hl=zh-cn#prmsn)、[service 权限](https://developer.android.com/guide/topics/manifest/service-element?hl=zh-cn#prmsn)；
- 代码：在运行时代码中注册，例如[`registerReceiver()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#registerReceiver\(android.content.BroadcastReceiver,%20android.content.IntentFilter\))。

不过，有时这些权限并非由设备上 APK 的清单中的相应 `<permission>` 标记定义。在这种情况下，这些权限称为孤立权限。这种情况可能是由多种原因造成的，例如：

- 清单更新与包含权限检查的代码之间可能不同步
- build 中可能未包含具有相应权限的 APK，或者包含的版本不正确
- 检查或清单中的权限名称可能拼写有误

恶意应用程序可以定义并获取一个孤立的权限。如果这种情况发生，那么信任该孤立权限以保护组件的特权应用程序就可能受到威胁。

如果享有特权的应用程序使用权限来保护或限制任何组件，这可能会使恶意应用程序获得对该组件的访问权限。例如，启动受权限保护的 activity、访问 content provider，或向受孤立权限保护的广播接收器进行广播。

它还可能造成这样一种情况：具有特权的应用程序被欺骗，误以为恶意应用程序是合法应用程序，从而加载文件或内容。

### 缓解措施

请确保您的应用用于保护组件的所有自定义权限也已在您的清单文件中定义。

该应用使用自定义权限 `my.app.provider.READ` 和 `my.app.provider.WRITE` 来保护对 content provider 的访问权限：

[Xml](https://developer.android.com/privacy-and-security/risks/custom-permissions?hl=zh-cn#xml)

```xml
<provider android:name="my.app.database.CommonContentProvider" android:readPermission="my.app.provider.READ" android:writePermission="my.app.provider.WRITE" android:exported="true" android:process=":myappservice" android:authorities="my.app.database.contentprovider"/>
```

该应用还会定义和使用这些自定义权限，从而防止其他恶意应用这样做：

[Xml](https://developer.android.com/privacy-and-security/risks/custom-permissions?hl=zh-cn#xml)

```xml
<permission android:name="my.app.provider.READ"/>
<permission android:name="my.app.provider.WRITE"/>
<uses-permission android:name="my.app.provider.READ" />
<uses-permission android:name="my.app.provider.WRITE" />
```

---

## 风险：滥用 android:protectionLevel

此属性用于描述权限的潜在风险级别，并指明系统在决定是否授予权限时应遵循的流程。

### 缓解措施

#### 避免使用“正常”或“危险”保护级别

在权限设置中使用普通或危险级别的保护意味着大多数应用都可以请求并获得权限：

- “normal”只需声明即可
- “dangerous”将会得到许多用户的批准

因此，这些 `protectionLevels` 的安全性很低。

#### 使用签名权限（Android 10 或更高版本）

尽可能使用签名保护级别。使用此功能可确保只有与创建权限的应用使用相同证书签名的其他应用才能访问这些受保护功能。确保您使用的是专用（不重复使用的）签名证书，并将其安全地存储在[密钥库](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn)中。

在清单中按如下方式定义自定义权限：

[Xml](https://developer.android.com/privacy-and-security/risks/custom-permissions?hl=zh-cn#xml)

```xml
<permission
    android:name="my.custom.permission.MY_PERMISSION"
    android:protectionLevel="signature"/>
```

限制对某个活动等内容的访问权限，仅允许已获得以下自定义权限的应用访问：

[Xml](https://developer.android.com/privacy-and-security/risks/custom-permissions?hl=zh-cn#xml)

```xml
<activity android:name=".MyActivity" android:permission="my.custom.permission.MY_PERMISSION"/>
```

任何其他使用与声明此自定义权限的应用相同的证书签名的应用都将被授予对 .MyActivity activity的访问权限，并且需要在其 Manifest 文件中按如下方式声明：

[Xml](https://developer.android.com/privacy-and-security/risks/custom-permissions?hl=zh-cn#xml)

```xml
<uses-permission android:name="my.custom.permission.MY_PERMISSION" />
```

#### 谨防签名自定义权限（Android 版本低于 10）

如果您的应用以 Android 10 以下版本为目标平台，那么每当您的应用的自定义权限因卸载或更新而被移除时，恶意应用都可能仍能使用这些自定义权限，从而绕过检查。这是由于存在一个提权漏洞 ([`CVE-2019-2200`](https://nvd.nist.gov/vuln/detail/CVE-2019-2200))，该漏洞已在 Android 10 中[修复](https://source.android.com/docs/security/bulletin/2020-02-01?hl=zh-cn)。

这是建议签名检查而非自定义权限的原因之一（同时也存在竞态条件的风险）。

---

## 风险：竞态条件

如果合法应用 `A` 定义了一个由其他 `X` 应用使用的签名自定义权限，但该权限随后被卸载，那么恶意应用 `B` 可以使用不同的 `protectionLevel`（例如_normal_）定义该相同的自定义权限。这样一来，`B` 便可以访问 `X` 应用中受该自定义权限保护的所有组件，而无需使用与应用 `A` 相同的证书为任何组件签名。

如果 `B` 在 `A` 之前安装，也会发生同样的情况。

### 缓解措施

如果您希望某个组件仅适用于与提供该组件的应用具有相同签名的应用，则或许可以避免定义自定义权限来限制对该组件的访问。在这种情况下，您可以使用签名检查。如果您的某个应用向您的另一个应用发出请求，后者会先验证两者是否使用相同的证书进行签名，只有证书相同时才会遵照该请求行事。

---

## 资源

- [尽量减少权限请求](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)
- [权限概览](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn)
- [保护等级说明](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)
- [CustomPermissionTypo Android lint](https://cs.android.com/android-studio/platform/tools/base/+/mirror-goog-studio-main:lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks/PermissionErrorDetector.kt?hl=zh-cn)
- [如何使用 Android lint](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-main/docs/lint_guide.md#tips)
- [研究论文，深入介绍了 Android 权限和有趣的模糊测试发现](https://diaowenrui.github.io/paper/oakland21-li.pdf)
