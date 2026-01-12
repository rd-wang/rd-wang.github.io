---
title: Android-安全性-安全风险-不安全的反序列化
date: 2025-12-26 08:43:36 +0800
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
description: 在正常情况下，系统会序列化数据，然后再反序列化，而无需用户干预。不过，反序列化进程与其预期对象之间的信任关系可能会被恶意攻击者滥用，例如，他们可能会拦截和修改序列化对象。这会使恶意行为者能够执行拒绝服务 (DoS)、特权提升和远程代码执行 (RCE) 等攻击。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

在存储或传输大量 Java 对象数据时，通常更高效的做法是先序列化数据。然后，接收数据的application, activity, 或 provider将对数据进行反序列化过程，最终处理数据。正常情况下，数据会在没有任何用户干预的情况下进行序列化和反序列化。不过，反序列化进程与其预期对象之间的信任关系可能会被恶意攻击者滥用，例如，恶意行为者可以拦截和更改序列化对象。这将使恶意行为者能够执行拒绝服务 (DoS)、权限提升和远程代码执行 (RCE) 等攻击。

虽然 [`Serializable`](https://developer.android.com/reference/java/io/Serializable?hl=zh-cn) 类是管理序列化的一个常用方法，但 Android 还有自己的一个用于处理序列化的类，称为 [`Parcel`](https://developer.android.com/reference/android/os/Parcel?hl=zh-cn)。借助 `Parcel` 类，可以将对象数据序列化为字节流数据，并使用 [`Parcelable`](https://developer.android.com/reference/android/os/Parcelable?hl=zh-cn) 接口打包到 `Parcel` 中。这样可以更高效地运输或存储 `Parcel`。

不过，在使用 `Parcel` 类时应慎重考虑，因为它旨在成为一种高效的 IPC 传输机制，但不应用于在本地永久性存储空间中存储序列化对象，因为这可能会导致数据兼容性问题或数据丢失。需要读取数据时，可以使用 `Parcelable` 接口将 `Parcel` 反序列化，并将其转换回对象数据。

Android 系统中存在三种主要的反序列化漏洞利用途径：

- 利用开发者对反序列化自定义类类型对象安全性的错误认知。实际上，任何类所获取的任何对象都可能被替换为恶意内容，最坏的情况下，可能会干扰同一应用程序或其他应用程序的类加载器。这种干扰表现为注入危险值，根据类别的目的，这些危险值可能导致数据泄露或账户接管等后果。
- 利用设计上被认为不安全的反序列化方法（例如 [CVE-2023-35669](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2023-35669)，一个本地权限提升漏洞，允许通过深度链接反序列化向量注入任意 JavaScript 代码。）。
- 利用应用程序逻辑中的缺陷（例如  [CVE-2023-20963](https://nvd.nist.gov/vuln/detail/CVE-2023-20963)，这是一个本地权限提升缺陷，允许应用程序通过 Android WorkSource parcel 逻辑中的缺陷在特权环境中下载和执行代码）。

## 影响

任何反序列化不可信或恶意序列化数据的应用，都可能容易受到远程代码执行或拒绝服务攻击。

## 风险：不受信任的输入反序列化

攻击者可以利用应用逻辑中缺少 parcel 验证，注入任意对象。这些对象在反序列化后可能会强制应用执行恶意代码，从而可能导致拒绝服务 (DoS)、权限提升和远程代码执行 (RCE)。

这类攻击可能非常隐蔽。例如，一个应用程序可能包含一个 intent，该 intent 仅期望一个参数，该参数在验证后将被反序列化。如果攻击者除了预期的参数外，还发送了第二个extra的恶意额外参数，这将导致注入的所有数据对象被反序列化，因为 intent 会将 extra 视为 [`Bundle`](https://developer.android.com/reference/android/os/Bundle?hl=zh-cn)。恶意用户可能会利用这种行为注入对象数据，一旦反序列化，就可能导致远程代码执行、数据泄露或数据丢失。

### 缓解措施

最佳实践是假定所有序列化数据不可信且可能具有恶意。为确保序列化数据的完整性，请对数据执行验证检查，确保其是应用预期的正确类和格式。

一个可行的解决方案是，为 `java.io.ObjectInputStream` [库](https://developer.android.com/reference/java/io/ObjectInputStream?hl=zh-cn)实现预测模式。通过修改负责反序列化的代码，您可以确保在 intent 中仅反序列化[明确指定的一组类](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html#harden-your-own-javaioobjectinputstream)。

从 Android 13（API 级别 33）开始，`Intent` 类中更新了多种方法，这些方法被视为处理软件包的旧方法（现已废弃）的更安全替代方案。这些类型更安全的新方法（例如 [`getParcelableExtra(java.lang.String, java.lang.Class)`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#getParcelableExtra\(java.lang.String,%20java.lang.Class%3CT%3E\)) 和 [`getParcelableArrayListExtra(java.lang.String, java.lang.Class)`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#getParcelableArrayListExtra\(java.lang.String,%20java.lang.Class%3C?%20extends%20T%3E\))）会执行数据类型检查，以发现可能导致应用程序崩溃并可能被利用来执行权限提升攻击的数据类型不匹配漏洞。（例如 [CVE-2021-0928](https://nvd.nist.gov/vuln/detail/CVE-2021-0928)）。

以下示例演示了如何实现安全版本的 `Parcel` 类：

假设类 `UserParcelable` 实现了 `Parcelable`，并创建了一个用户数据实例，然后将其写入 `Parcel`。然后，您可以使用 [`readParcelable`](https://developer.android.com/reference/android/os/Parcel?hl=zh-cn#readParcelable\(java.lang.ClassLoader,%20java.lang.Class%3CT%3E\)) 的以下类型更安全的方法来读取序列化文件包：

[Kotlin](https://developer.android.com/privacy-and-security/risks/unsafe-deserialization?hl=zh-cn#kotlin)
```kotlin
val parcel = Parcel.obtain()
val userParcelable = parcel.readParcelable(UserParcelable::class.java.classLoader)
```
[Java](https://developer.android.com/privacy-and-security/risks/unsafe-deserialization?hl=zh-cn#java)

```java
Parcel parcel = Parcel.obtain();
UserParcelable userParcelable = parcel.readParcelable(UserParcelable.class, UserParcelable.CREATOR);
```

请注意，在上面的 Java 示例中，方法中使用了 `UserParcelable.CREATOR`。这个必需参数用于告知 `readParcelable` 方法需要什么类型，并且性能比现已弃用的 `readParcelable` 方法版本更高。

## 具体风险

本节收集了需要非标准缓解策略或在某些 SDK 级别中已缓解的风险，此处仅供参考。

### 风险：不需要的对象反序列化

在类中实现 `Serializable` 接口会自动导致该类的所有子类型实现该接口。在这种情况下，某些对象可能会继承上述接口，这意味着一些原本不应该被反序列化的对象仍然会被处理。这可能会无意中扩大攻击面。

#### 缓解措施

如果某个类继承了 `Serializable` 接口，则根据 [OWASP 指南](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html#prevent-deserialization-of-domain-objects)，应按如下方式实现 `readObject` 方法，以避免类中的一组对象被反序列化：

[Kotlin](https://developer.android.com/privacy-and-security/risks/unsafe-deserialization?hl=zh-cn#kotlin)
```kotlin
@Throws(IOException::class)
private final fun readObject(in: ObjectInputStream) {
    throw IOException("Cannot be deserialized")
}
```
[Java](https://developer.android.com/privacy-and-security/risks/unsafe-deserialization?hl=zh-cn#java)

```java
private final void readObject(ObjectInputStream in) throws java.io.IOException {
    throw new java.io.IOException("Cannot be deserialized");
}
```

## 资源

- [Parcelable](https://developer.android.com/reference/android/os/Parcel?hl=zh-cn#parcelables)
- [包裹](https://developer.android.com/reference/android/os/Parcel?hl=zh-cn)
- [Serializable](https://developer.android.com/reference/java/io/Serializable?hl=zh-cn)
- [Intent](https://developer.android.com/reference/android/content/Intent?hl=zh-cn)
- [Android 反序列化漏洞：简要历史记录](https://securitylab.github.com/research/android-deserialization-vulnerabilities/)
- [Android 软件包：优点、缺点和改进建议（视频）](https://www.youtube.com/watch?v=qIzMKfOmIAA&hl=zh-cn)
- [Android 软件包：优点、缺点和改进建议（演示文稿幻灯片）](https://i.blackhat.com/EU-22/Wednesday-Briefings/EU-22-Ke-Android-Parcels-Introducing-Android-Safer-Parcel.pdf)
- [CVE-2014-7911：使用 ObjectInputStream 提升 Android 5.0 以下的权限](https://seclists.org/fulldisclosure/2014/Nov/51)
- [CVE-CVE-2017-0412](https://bugs.chromium.org/p/project-zero/issues/detail?id=1002)
- [CVE-2021-0928：Parcel 序列化/反序列化不匹配](https://nvd.nist.gov/vuln/detail/CVE-2021-0928)
- [OWASP 指南](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html#prevent-deserialization-of-domain-objects)

