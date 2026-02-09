---
title: Android-权限-声明应用权限
date: 2026-2-9 16:14:46 +0800
categories:
  - Android
  - Permissions
  - Declare
tags:
  - Android
  - Permissions
  - Declare
description: 如果您的应用请求应用权限，则必须在应用的清单文件中声明这些权限。
math: true
---
# 声明应用权限

如[权限使用工作流程](https://developer.android.com/training/basics/permissions?hl=zh-cn#workflow)所述，如果您的应用请求应用权限，则必须在应用的清单文件中声明这些权限。这些声明有助于应用商店和用户了解您的应用可能需要的权限。 

请求权限的过程取决于权限类型：

- 如果是[安装时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#install-time)（例如普通权限或签名权限），系统会在安装您的应用时自动为其授予相应权限。
- 如果是[运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)或[特殊权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#special)，并且您的应用安装在搭载 Android 6.0（API 级别 23）或更高版本的设备上，则您必须自己请求[运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn)或[特殊权限](https://developer.android.com/training/permissions/requesting-special?hl=zh-cn)。

>**注意**：请仔细考虑要在应用清单中声明哪些权限。请仅添加您的应用需要的权限。对于应用请求的每项权限，请确保它能为用户带来明显的益处，并且请求方式对用户来说清楚明晰。

## 向应用清单添加声明

如需声明应用可能请求的权限，请在应用的清单文件中添加相应的 [`<uses-permission>`](https://developer.android.com/guide/topics/manifest/uses-permission-element?hl=zh-cn) 元素。例如，如果应用需要访问相机，则应在 `AndroidManifest.xml` 中添加以下代码行：

```xml
<manifest ...>
    **<uses-permission android:name="android.permission.CAMERA"/>**
    <application ...>
        ...
    </application>
</manifest>
```
## 将硬件声明为可选

某些权限（例如 [`CAMERA`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#CAMERA)）允许您的应用访问仅部分安卓设备才具备的硬件。如果您的应用声明了这些与[硬件相关权限](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=zh-cn#permissions-features)之一，请考虑您的应用是否还能在没有该硬件的设备上运行。在大多数情况下，硬件是可选的，因此最好在 [`<uses-feature>`](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=zh-cn) 声明中将 `android:required` 设置为 `false`，从而将硬件声明为可选项，如 `AndroidManifest.xml` 文件中的以下代码段所示：

```xml
<manifest ...>
    <application>
        ...
    </application>
    **<uses-feature android:name="android.hardware.camera"
                  android:required="false"** />
<manifest>
```

> **注意** ：如果您没有在 `<uses-feature>` 中将 `android:required` 设置为 `false` ，Android 会假定您的应用必须在有该硬件的情况下才能运行。 随后，系统会[阻止某些设备安装您的 app](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=zh-cn#market-feature-filtering)。

### 确定硬件可用性

如果将硬件声明为可选，则您的应用程序有可能在没有该硬件的设备上运行。如需检查设备是否具有特定的硬件，请使用 [`hasSystemFeature()`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#hasSystemFeature\(java.lang.String\)) 方法，如以下代码段所示。如果设备不具有该硬件，只需在您的应用中停用此功能即可。

[Kotlin](https://developer.android.com/training/permissions/declaring?hl=zh-cn#kotlin)
```kotlin
// Check whether your app is running on a device that has a front-facing camera.
if (applicationContext.packageManager.hasSystemFeature(
        PackageManager.FEATURE_CAMERA_FRONT)) {
    // Continue with the part of your app's workflow that requires a
    // front-facing camera.
} else {
    // Gracefully degrade your app experience.
}
```
[Java](https://developer.android.com/training/permissions/declaring?hl=zh-cn#java)

```java
// Check whether your app is running on a device that has a front-facing camera.
if (getApplicationContext().getPackageManager().hasSystemFeature(
        PackageManager.FEATURE_CAMERA_FRONT)) {
    // Continue with the part of your app's workflow that requires a
    // front-facing camera.
} else {
    // Gracefully degrade your app experience.
}
```
## 按 API 级别声明权限

如需仅针对支持运行时权限的设备（即搭载 Android 6.0 [API 级别 23] 或更高版本的设备）声明某项权限，请添加 [`<uses-permission-sdk-23>`](https://developer.android.com/guide/topics/manifest/uses-permission-sdk-23-element?hl=zh-cn)（而非 [`<uses-permission>`](https://developer.android.com/guide/topics/manifest/uses-permission-element?hl=zh-cn)）元素。

使用这两个元素中的任意一个时，您都可以设置 `maxSdkVersion` 属性，指明搭载的 Android 版本高于指定值的设备不需要特定权限。这样，您就可以消除不必要的权限，同时仍为旧款设备提供兼容性。

例如，您的应用可能会显示用户在使用您的应用时创建的媒体内容，例如照片或视频。在这种情况下，只要您的应用是针对 Android 10 或更高版本开发的，就无需在搭载 Android 10（API 级别 29）或更高版本的设备上使用 [`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_EXTERNAL_STORAGE) 权限。不过，为了与旧款设备兼容，您可以声明 `READ_EXTERNAL_STORAGE` 权限，并将 `android:maxSdkVersion` 设置为 28。