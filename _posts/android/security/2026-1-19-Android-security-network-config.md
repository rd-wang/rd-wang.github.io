---
title: Android-安全性-网络安全配置
date: 2026-1-19 14:59:15 +0800
categories:
  - Android
  - Security
  - Network-Config
tags:
  - Android
  - Security
  - Network
  - Config
description: 网络安全配置功能允许您在安全、声明式的配置文件中自定义应用程序的网络安全设置，而无需修改应用程序代码。
math: true
---
网络安全配置功能允许您在安全、声明式的[配置文件](https://developer.android.com/training/articles/security-config?hl=zh-cn#FileFormat)中自定义应用程序的网络安全设置，而无需修改应用程序代码。这些设置可以针对特定域名和特定应用进行配置。此功能的主要功能包括：

- **自定义信任锚：** 自定义应用程序安全连接所信任的证书颁发机构 (CA)。例如，信任特定的自签名证书，或限制应用程序信任的公共 CA 集。
- **仅调试替换：** 在不增加已安装应用风险的情况下，安全地调试应用中的安全连接。
- **选择停用明文流量：** 防止应用意外使用明文流量。
- **证书透明度启用：** 限制应用的安全连接仅使用可验证已记录日志的证书。
- **证书绑定：** 限制应用的安全连接仅使用特定证书。

## 添加网络安全配置文件

网络安全配置功能使用 XML 文件来指定应用程序的设置。您必须在应用程序的清单文件中添加指向此文件的条目。以下代码片段摘自清单文件，演示了如何创建此条目：
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config"
                    ... >
        ...
    </application>
</manifest>
```

## 自定义可信 CA

您可能希望您的应用信任一组自定义的证书颁发机构 (CA)，而不是平台默认的 CA。最常见的原因包括：

- 连接到具有自定义 CA 的主机，例如自签名 CA 或公司内部颁发的 CA。
- 将 CA 集限制为您信任的 CA，而不是所有预装的 CA。
- 信任系统中未包含的其他 CA。

默认情况下，所有应用程序的安全连接（使用 TLS 和 HTTPS 等协议）都信任预装的系统 CA，而面向 Android 6.0（API 级别 23）及更低版本的应用程序默认也信任用户添加的 CA 存储。您可以使用 base-config（用于应用程序范围的自定义）或 domain-config（用于每个域的自定义）来自定义应用程序的连接。

### 配置自定义 CA

您可能希望连接到使用自签名 SSL 证书的主机，或者连接到其 SSL 证书由您信任的非公共 CA（如公司的内部 CA）颁发的主机。 以下代码段演示了如何在 `res/xml/network_security_config.xml` 中为应用配置自定义 CA：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <trust-anchors>
            <certificates src="@raw/my_ca"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

将自签名或非公开的 CA 证书（PEM 或 DER 格式）添加到 `res/raw/my_ca`。

### 限制受信任的证书颁发机构 (CA) 的数量

如果您不希望应用信任系统信任的所有证书颁发机构 (CA)，您可以指定一个精简的受信任 CA 集合。这样可以保护应用免受其他 CA 颁发的欺诈性证书的侵害。

限制可信 CA 集的配置与针对特定网域[信任自定义 CA](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#ConfigCustom) 相似，区别在于资源中提供了多个 CA。以下代码段演示了如何在 `res/xml/network_security_config.xml` 中限制应用的可信 CA 集：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">secure.example.com</domain>
        <domain includeSubdomains="true">cdn.example.com</domain>
        <trust-anchors>
            <certificates src="@raw/trusted_roots"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

将受信任的 CA 证书（PEM 或 DER 格式）添加到 `res/raw/trusted_roots` 目录中。请注意，如果使用 PEM 格式，该文件必须仅包含 PEM 数据，不得包含任何额外文本。您也可以提供多个 [`<certificates>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#certificates) 元素，而不是单个。
### 信任其他 CA 

您可能希望您的应用信任系统未信任的其他 CA，例如，如果系统尚未包含该 CA，或者该 CA 不符合包含在 Android 系统中的要求。您可以使用类似以下摘录的代码，在 `res/xml/network_security_config.xml` 中为配置指定多个证书源。

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="@raw/extracas"/>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
</network-security-config>
```
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="@raw/extracas"/>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
</network-security-config>

## 配置用于调试的 CA

调试通过 HTTPS 连接的应用时，您可能需要连接到没有为生产服务器提供 SSL 证书的本地开发服务器。为了支持此操作，而又不对应用的代码进行任何修改，您可以通过使用 `debug-overrides` 指定仅在 [android:debuggable](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#debug) 为 `true` 时才可信的仅调试 CA。通常，IDE 和构建工具会自动为非发布 build 设置此标记。

这比一般的条件代码更安全，因为出于安全考虑，应用商店不接受被标记为可调试的应用。

以下代码段展示了如何在 `res/xml/network_security_config.xml` 中指定仅用于调试的 CA：
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="@raw/debug_cas"/>
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="@raw/debug_cas"/>
        </trust-anchors>
    </debug-overrides>
</network-security-config>

## 选择加入证书透明度

> 注意：证书透明度支持仅适用于 Android 16（API 级别 36）及更高版本。

证书透明度（CT，RFC 6962）是一种旨在增强数字证书安全性的互联网标准。它要求 CA 将所有已颁发的证书提交到记录这些证书的公共日志中，从而提高证书颁发过程的透明度和问责制。

通过维护所有证书的可验证记录，CT 大大增加了恶意行为者伪造证书或 CA 错误颁发证书的难度。

这有助于保护用户免受中间人攻击和其他安全威胁。更多信息，请参阅  [transparency.dev](https://certificate.transparency.dev/)上的说明。有关 Android 上的 CT 合规性的更多信息，请参阅[Android's CT policy](https://developer.android.com/privacy-and-security/certificate-transparency-policy)。

默认情况下，无论证书是否已记录在 CT 日志中，都会被接受。为确保您的应用仅连接到证书已记录在 CT 日志中的目标，您可以选择[全局](https://developer.android.com/privacy-and-security/security-config#base-config)或者[按域](https://developer.android.com/privacy-and-security/security-config#domain-config) [启用此功能](https://developer.android.com/privacy-and-security/security-config#certificateTransparency)。
## 明文流量

开发者可以选择是否允许应用程序使用明文流量（即使用未加密的 HTTP 协议而非 HTTPS）。更多详情请参阅[`NetworkSecurityPolicy.isCleartextTrafficPermitted()`](https://developer.android.com/reference/android/security/NetworkSecurityPolicy#isCleartextTrafficPermitted\(\))

明文流量的默认行为取决于 API 级别：
- 在 Android 8.1（API 级别 27）及更早版本中，默认启用明文传输支持。应用程序可以选择 [禁用明文传输](https://developer.android.com/privacy-and-security/security-config#CleartextTrafficOptOut)以增强安全性。
- 从 Android 9（API 级别 28）开始，默认情况下禁用明文传输支持。需要明文传输的应用可以选择[启用明文传输](https://developer.android.com/privacy-and-security/security-config#CleartextTrafficOptIn).。
### 选择退出明文流量

>**注意：** 本节中的指导仅适用于面向 Android 8.1（API 级别 27）或更低版本的应用程序。

如果您希望您的应用仅使用安全连接连接到目标位置，您可以选择不支持向这些目标位置传输明文流量。此选项有助于防止由于后端服务器等外部来源提供的 URL 发生更改而导致应用程序出现意外退化。
例如，您可能希望您的应用程序确保与 secure.example.com 的连接始终通过 HTTPS 进行，

例如，您可能希望应用确保所有与 `secure.example.com` 的连接始终通过 HTTPS 完成，以保护敏感流量免受恶意网络的攻击。

以下代码段展示了如何在 `res/xml/network_security_config.xml` 中停用明文：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">secure.example.com</domain>
    </domain-config>
</network-security-config>
```

### 选择加入明文流量

>**注意：** 本节中的指导仅适用于面向 Android 9（API 级别 28）或更高版本的应用程序。

如果您的应用需要连接到使用明文流量（HTTP）的目标网站，您可以选择启用对明文流量的支持。 
例如，您可能希望允许您的应用与`insecure.example.com`建立不安全连接。
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">insecure.example.com</domain>
    </domain-config>
</network-security-config>
```

如果您的应用需要允许向任何域发送明文流量，请在 `base-config`中设置 `cleartextTrafficPermitted="true"`。请注意，应尽可能避免这种不安全的配置。
## 固定证书

一般情况下，应用信任所有预装 CA。如果有预装 CA 颁发了伪造证书，则应用将面临来自路径攻击者的风险。有些应用选择通过限制信任的 CA 集或通过固定证书来限制其接受的证书集。

证书绑定是通过提供一组基于公钥哈希值（X.509 证书的 `SubjectPublicKeyInfo`）的证书来实现的。只有当证书链包含至少一个已固定的公钥时，该证书链才是有效的。

请注意，在使用证书固定时，您应该始终包含一个备份密钥，这样，如果您被迫切换到新密钥或更改 CA（当固定到 CA 证书或该 CA 的中间证书时），您的应用程序的连接性就不会受到影响。否则，您必须推送应用更新以恢复连接。

此外，还可以为密码设置过期时间，过期后将不再执行密码绑定操作。这有助于防止未更新的应用程序出现连接问题。但是，为证书设置过期时间可能会使攻击者绕过您已固定的证书。

以下代码段展示了如何在 `res/xml/network_security_config.xml` 中固定证书：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set expiration="2018-01-01">
            <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
            <!-- backup pin -->
            <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

## 配置继承行为

未在特定配置中设置的值将被继承。这种行为允许更复杂的配置，同时保持配置文件的可读性。

例如，未在 `domain-config` 中设置的值将从父级 `domain-config`（如果已嵌套）或者 `base-config`（如果未嵌套）中获取。未在 `base-config` 中设置的值将使用平台默认值。

例如，假设所有连接到 `example.com` 子域的连接都必须使用一组自定义的证书颁发机构 (CA)。此外，除连接到 `secure.example.com` 外，允许向这些域发送明文流量。通过将 `secure.example.com` 的配置嵌套在 `example.com` 的配置中，无需重复配置 `trust-anchors`

以下代码段展示了此嵌套在 `res/xml/network_security_config.xml` 中的模式：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <trust-anchors>
            <certificates src="@raw/my_ca"/>
        </trust-anchors>
        <domain-config cleartextTrafficPermitted="false">
            <domain includeSubdomains="true">secure.example.com</domain>
        </domain-config>
    </domain-config>
</network-security-config>
```

## 配置文件格式

网络安全配置功能使用 XML 文件格式。 文件的整体结构如以下代码示例所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="..."/>
            ...
        </trust-anchors>
    </base-config>

    <domain-config>
        <domain>android.com</domain>
        ...
        <trust-anchors>
            <certificates src="..."/>
            ...
        </trust-anchors>
        <pin-set>
            <pin digest="...">...</pin>
            ...
        </pin-set>
    </domain-config>
    ...
    <debug-overrides>
        <trust-anchors>
            <certificates src="..."/>
            ...
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

以下部分将介绍语法与文件格式的其他详细信息。

### `<network-security-config>` 

可包含：
- 0 或 1 个 [`<base-config>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#base-config)  
- 任意数量的 [`<domain-config>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#domain-config)  
- 0 或 1 个 [`<debug-overrides>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#debug-overrides)

### `<base-config>`

语法：

```xml
<base-config cleartextTrafficPermitted=["true" | "false"]>
    ...
</base-config>
```

可包含：

- [`<trust-anchors>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#trust-anchors)  [<`certificateTransparency>`](https://developer.android.com/privacy-and-security/security-config#certificateTransparency)

说明：

目标不在 [`domain-config`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#domain-config) 涵盖范围内的所有连接所使用的默认配置。

未设置的任何值均使用平台默认值。

以 Android 9（API 级别 28）及更高版本为目标平台的应用的默认配置如下所示：
```xml
<base-config cleartextTrafficPermitted="false">
    <trust-anchors>
        <certificates src="system" />
    </trust-anchors>
</base-config>
```
<base-config cleartextTrafficPermitted="false">
    <trust-anchors>
        <certificates src="system" />
    </trust-anchors>
</base-config>

以 Android 7.0（API 级别 24）到 Android 8.1（API 级别 27）为目标平台的应用的默认配置如下所示：
```xml
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <certificates src="system" />
    </trust-anchors>
</base-config>
```
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <certificates src="system" />
    </trust-anchors>
</base-config>

以 Android 6.0（API 级别 23）及更低版本为目标平台的应用的默认配置如下所示：

<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <certificates src="system" />
        <certificates src="user" />
    </trust-anchors>
</base-config>
```xml
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <certificates src="system" />
        <certificates src="user" />
    </trust-anchors>
</base-config>
```
### `<domain-config>`

语法：
```xml
<domain-config cleartextTrafficPermitted=["true" | "false"]>
    ...
</domain-config>
```

可包含：

- 1 个或更多 [`<domain>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#domain)  
- 0 或 1 个 [`<trust-anchors>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#trust-anchors)  
- 0 或 1 个 [`<pin-set>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#pin-set)  
- 任意数量的嵌套 `<domain-config>`

说明：

用于连接到特定目标的配置，由`domain` 元素定义。

请注意，如果多个 `domain-config` 元素涵盖同一目标，则使用匹配域规则最具体（最长）的配置。
### `<domain>`

语法：

```xml
<domain includeSubdomains=["true" | "false"]>example.com</domain>
```


属性：

`includeSubdomains`

如果为 `"true"`，则此域名规则匹配该域名及其所有子域名，包括子域名的子域名。否则，该规则仅适用于完全匹配。

### `<certificateTransparency>`

语法：
```xml
<certificateTransparency enabled=["true" | "false"]/>
```

说明：

如果为 `true`,则该应用程序将使用证书透明度日志来验证证书。当应用程序使用自己的证书（或用户存储的证书）时，该证书很可能不是公开的，因此无法使用证书透明度进行验证。 默认情况下，这些情况下的验证功能处于禁用状态。但仍然可以通过在域配置中使用 `<certificateTransparency enabled="true"/>` 来强制启用验证。对于每个 [`<domain-config>`](https://developer.android.com/privacy-and-security/security-config#domain-config)，评估过程遵循以下顺序：

1. 如果启用了 `certificateTransparency` ，则启用验证。
2. 如果任何 [`<trust-anchors>`](https://developer.android.com/privacy-and-security/security-config#trust-anchors) 为 `"user"` 或内联（即“@raw/cert.pem”），则禁用验证。
3. 否则，就沿用[继承的配置](https://developer.android.com/privacy-and-security/security-config#ConfigInheritance).
### `<debug-overrides>`

语法：
```xml
<debug-overrides>
    ...
</debug-overrides>
```

可包含：

- 0 or 1 个 [`<trust-anchors>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#trust-anchors)

说明：

在 [android:debuggable](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#debug) 为 `"true"` 时将应用的替换，IDE 和构建工具生成的非发布 build 通常属于此情况。在 `debug-overrides` 中指定的信任锚点将添加到所有其他配置中，并且当服务器的证书链使用这些仅用于调试的信任锚点之一时，不会执行证书固定。如果 [android:debuggable](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn#debug) 为 `"false"`，则完全忽略此部分。

### `<trust-anchors>`

语法：

```xml
<trust-anchors>
...
</trust-anchors>
```

可包含：

- 任意数量的 [`<certificates>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#certificates)

说明：

用于安全连接的信任锚集。

### `<certificates>`

语法：

```xml
<certificates src=["system" | "user" | "raw resource"]
              overridePins=["true" | "false"] />
```

说明：

用于 `trust-anchors` 元素的 X.509 证书集。

属性：

`src`

CA 证书的来源。每个证书可以是以下项之一：

- 指向包含 X.509 证书的文件的原始资源 ID。 证书必须以 DER 或 PEM 格式编码。如果为 PEM 证书，则文件不得包含额外的非 PEM 数据，例如注释。
- `"system"`，表示预装系统 CA 证书
- `"user"`，表示用户添加的 CA 证书

`overridePins`

指定此来源的 CA 是否绕过证书固定。如果为 `"true"`，则不会对由此来源的某个 CA 签名的证书链执行绑定。这对于调试 CA 或测试针对应用程序安全流量的中间人攻击非常有用。

默认值为 `"false"`，除非在 `debug-overrides` 元素中另外指定（在这种情况下，默认值为 `"true"`。）

### `<pin-set>`

语法：

```xml
<pin-set expiration="date">
...
</pin-set>
```

可包含：

- 任意数量的 [`<pin>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#pin)

说明：

一组公钥固定。为了确保安全连接的可信度，信任链中的某个公钥必须位于这组 PIN 码中。PIN 码的格式请参见 [`<pin>`](https://developer.android.com/privacy-and-security/security-config?hl=zh-cn#pin)。

属性：

`expiration`

采用 `yyyy-MM-dd` 格式的日期，用于指定密码失效日期，失效后密码将无法使用。如果未设置此属性，则密码不会失效。

过期功能有助于防止应用程序在未更新其 PIN 码设置时出现连接问题，例如用户禁用应用程序更新时。

### `<pin>`

语法：

```xml
<pin digest=["SHA-256"]>base64 encoded digest of X.509
    SubjectPublicKeyInfo (SPKI)</pin>
```

属性：

`digest`

用于生成证书固定的摘要算法。目前仅支持 `"SHA-256"`。

## 其他资源

如需详细了解网络安全配置，请参阅以下资源。

### Codelab

- [Android 网络安全配置](https://developer.android.com/codelabs/android-network-security-config?hl=zh-cn)

