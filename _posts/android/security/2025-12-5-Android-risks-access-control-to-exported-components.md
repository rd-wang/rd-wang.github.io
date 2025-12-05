---
title: Android-安全风险-对导出的组件的基于权限的访问控制
date: 2025-12-5 15:34:26 +0800
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
description: 导出易受攻击的组件可能被用于获取敏感资源或执行敏感操作。这种恶意行为的影响取决于易受攻击组件的具体情况及其权限。
math: true
---
# 对导出组件的基于权限的访问控制

**OWASP 类别**： [MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

Android 权限是应用清单中声明的字符串标识符，用于请求访问受限数据或执行受限操作，由 Android 框架在运行时强制执行。

[Android 权限级别](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)表示与权限相关的潜在风险：

- **普通**：低风险权限，会在安装时自动授予
- **危险**：可能允许访问敏感用户数据的高风险权限，需要在运行时获得用户明确批准
- **签名**：仅授予使用与声明权限的应用相同的证书签名的应用，通常用于系统应用或同一开发者的应用之间的互动

当应用的组件（例如[Activity](https://developer.android.com/reference/android/app/Activity)、[Receiver](https://developer.android.com/privacy-and-security/security-tips#broadcast-receivers)、[Content Provider](https://developer.android.com/privacy-and-security/security-tips#content-providers)或 [Service](https://developer.android.com/privacy-and-security/security-tips#services)）满足以下所有条件时，就会出现与基于权限的访问控制相关的漏洞：

- 组件未与 `Manifest` 中的任何 `android:permission` 相关联；
- 组件执行敏感任务，并且用户已批准相应权限；
- 组件已导出；
- 该组件不会执行任何手动（清单级或代码级）权限检查；

在这种情况下，恶意应用可以滥用易受攻击组件的权限，将易受攻击应用的权限代理给恶意应用，从而执行敏感操作。

## 影响

导出易受攻击的组件可能被用于获取敏感资源或执行敏感操作。这种恶意行为的影响取决于易受攻击组件的具体情况及其权限。

## 缓解措施

### 需要敏感任务的权限

导出具有敏感权限的组件时，应要求所有传入请求都拥有相同的权限。Android Studio IDE 内置了针对 [Receiver](https://googlesamples.github.io/android-custom-lint-rules/checks/ExportedReceiver.md.html)和[Service](https://googlesamples.github.io/android-custom-lint-rules/checks/ExportedService.md.html)的 lint 检查，可以发现此漏洞并建议请求相应的权限。

开发者可以通过在 `Manifest` 文件中声明权限，或在实现服务时在代码级别声明权限，来要求传入请求具有相应权限，如以下示例所示。

[Xml](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components?hl=zh-cn#xml)
```xml
<manifest ...>
    <uses-permission android:name="android.permission.READ_CONTACTS" />

    <application ...>
        <service android:name=".MyExportService"
                 android:exported="true"
                 android:permission="android.permission.READ_CONTACTS" />

        </application>
</manifest>
```
[Kotlin](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components?hl=zh-cn#kotlin)
```kotlin
class MyExportService : Service() {

    private val binder = MyExportBinder()

    override fun onBind(intent: Intent): IBinder? {
        // Enforce calling app has the required permission
        enforceCallingPermission(Manifest.permission.READ_CONTACTS, "Calling app doesn't have READ_CONTACTS permission.")
        // Permission is enforced, proceed with export logic
        return binder
    }

    // Inner class for your Binder implementation
    private inner class MyExportBinder : Binder() {
        // Permission is enforced, proceed with export logic
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components?hl=zh-cn#java)

```java
public class MyExportService extends Service {

    @Override
    public IBinder onBind(Intent intent) {
        // Enforce calling app has the required permission
        enforceCallingPermission(Manifest.permission.READ_CONTACTS, "Calling app doesn't have READ_CONTACTS permission.");

        return binder;

    }

    // Inner class for your Binder implementation
    private class MyExportBinder extends Binder {
        // Permission is enforced, proceed with export logic

    }
}
```

### 不导出组件

除非绝对必要，否则请避免导出有权访问敏感资源的组件。为此，您可以将 `Manifest` 文件中的 `android:exported` 设置为组件的 `false`。从 [API 级别 31](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn#exported) 及更高版本开始，此属性默认设为 `false`。

[Xml](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components?hl=zh-cn#xml)

```xml
<activity
    android:name=".MyActivity"
    android:exported="false"/>
```

### 采用基于签名的权限

当您在受您控制或归您所有的两个应用之间共享数据时，请使用基于签名的权限。此类权限不需要用户确认，而是会检查访问数据的应用是否使用相同的签名密钥进行了签名。这种设置可提供更流畅、更安全的用户体验。如果您声明了自定义权限，请务必遵守[相应的安全准则](https://developer.android.com/privacy-and-security/risks)。

[Xml](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components?hl=zh-cn#xml)

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <permission android:name="my_custom_permission_name"
                android:protectionLevel="signature" />
```

### 单任务端点

遵循[关注点分离](https://developer.android.com/topic/architecture?hl=zh-cn#separation-of-concerns)设计原则来实现应用。每个端点都应该只执行少量特定任务，并拥有特定的权限。这种良好的设计实践还允许开发者为每个端点应用细粒度的权限控制。例如，请避免创建一个同时提供日历和通讯录服务的端点。

## 资源

- [“安全保护”博客中的 Android 访问受应用保护的组件](https://blog.oversecured.com/Android-Access-to-app-protected-components/)
- [Content Provider 最佳实践](https://developer.android.com/privacy-and-security/security-tips?hl=zh-cn#content-providers)
- [运行时（危险）权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)
- [关注点分离设计原则](https://developer.android.com/topic/architecture?hl=zh-cn#separation-of-concerns)
- [Android 权限文档](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)
- [Android 广播接收器安全提示](https://developer.android.com/privacy-and-security/security-tips?hl=zh-cn#broadcast-receivers)
- [Android 服务安全提示](https://developer.android.com/privacy-and-security/security-tips?hl=zh-cn#services)
- [Android 12（API 31）导出的默认值设为“false”](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn#exported)
- [Lint 检查：导出的 PreferenceActivity 不应导出](https://googlesamples.github.io/android-custom-lint-rules/checks/ExportedPreferenceActivity.md.html)
- [lint 检查：导出的接收器不需要权限](https://googlesamples.github.io/android-custom-lint-rules/checks/ExportedReceiver.md.html)
- [lint 检查：导出的服务不需要权限](https://googlesamples.github.io/android-custom-lint-rules/checks/ExportedService.md.html)

