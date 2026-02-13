---
title: Android-权限-解释对敏感信息的访问
date: 2026-2-13 11:44:34 +0800
categories:
  - Android
  - Permissions
  - Explain
tags:
  - Android
  - Permissions
  - Explain
description: 与位置、麦克风和摄像头相关的权限使您的应用能够访问用户的特别敏感信息。该平台包含本页所述的多种机制，以帮助用户随时了解并控制哪些应用程序可以访问位置、麦克风和摄像头。
math: true
---
与位置、麦克风和摄像头相关的权限使您的应用能够访问用户的特别敏感信息。该平台包含本页所述的多种机制，以帮助用户随时了解并控制哪些应用程序可以访问位置、麦克风和摄像头。

只要您[遵循隐私最佳实践](https://developer.android.com/privacy/best-practices?hl=zh-cn)，这些保护隐私的系统功能就不会影响您的应用处理与位置、麦克风和摄像头相关的权限的方式。


具体来说，请确保在对应用执行相关操作时做到以下几点：

- 等到用户向您的应用授予 [`CAMERA`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#CAMERA) 权限后再使用设备的相机。
- 等到用户向您的应用授予 [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#RECORD_AUDIO) 权限后再使用设备的麦克风。
- 等到用户与您应用中某项需要获取位置信息的功能互动后再请求 [`ACCESS_COARSE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_COARSE_LOCATION) 权限或 [`ACCESS_FINE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_FINE_LOCATION) 权限，如介绍如何[请求位置信息权限](https://developer.android.com/training/location/permissions?hl=zh-cn#request-location-access-runtime)的指南中所述。
- 等到用户向您的应用授予 `ACCESS_COARSE_LOCATION` 权限或 `ACCESS_FINE_LOCATION` 权限后再请求 [`ACCESS_BACKGROUND_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_BACKGROUND_LOCATION) 权限。

## 隐私信息中心

在搭载 Android 12 或更高版本的受支持设备上，系统设置中会显示“隐私信息中心”屏幕。在此屏幕上，用户可以访问一些单独的屏幕，这些屏幕显示了应用何时访问位置信息、相机和麦克风信息。每个屏幕都会显示一个时间轴，指明不同的应用何时访问过特定类型的数据。图 1 显示了位置信息的数据访问时间轴。
![垂直时间轴显示了访问过位置信息的不同应用，以及访问发生的时间](https://rd-wang.github.io/assets/img/android/permissions/privacy-dashboard.svg)

**图 1.** “位置信息使用情况”屏幕，“隐私信息中心”的一部分。

### 显示数据访问的理由

您的应用可以向用户提供一个理由，帮助他们了解为什么您的应用访问位置信息、相机或麦克风信息。此理由可以显示在新的“隐私信息中心”屏幕和/或您应用的权限屏幕上。

如需解释为什么您的应用访问位置信息、相机和麦克风信息，请完成以下步骤：

1. 添加一个 activity，它在启动后会提供某种理由，说明为什么您的应用执行特定类型的数据访问操作。在此 activity 中，将 [`android:permission`](https://developer.android.com/guide/topics/manifest/activity-element?hl=zh-cn#prmsn) 属性设置为 [`START_VIEW_PERMISSION_USAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#START_VIEW_PERMISSION_USAGE)。
    
    如果您的应用以 Android 12 或更高版本为目标平台，您必须明确地[为 `android:exported` 属性定义一个值](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn#exported)。
    
2. 向新添加的 activity 添加以下 intent 过滤器：
```xml
    <!-- android:exported required if you target Android 12. -->
    <activity android:name=".DataAccessRationaleActivity"
              android:permission="android.permission.START_VIEW_PERMISSION_USAGE"
              android:exported="true">
           <!-- VIEW_PERMISSION_USAGE shows a selectable information icon on
                your app permission's page in system settings.
                VIEW_PERMISSION_USAGE_FOR_PERIOD shows a selectable information
                icon on the Privacy Dashboard screen. -->
        <intent-filter>
           <action android:name="android.intent.action.VIEW_PERMISSION_USAGE" />
           <action android:name="android.intent.action.VIEW_PERMISSION_USAGE_FOR_PERIOD" />
           <category android:name="android.intent.category.DEFAULT" />
           ...
        </intent-filter>
    </activity>
```

3. 决定数据访问理由活动应显示哪些内容。例如，您可以显示应用程序的网站或帮助中心文章。为了更详细地说明您的应用访问的数据类型以及访问发生的时间，请处理系统在调用权限使用意图时包含的额外信息：
    
    - 如果系统调用 `ACTION_VIEW_PERMISSION_USAGE`，您的应用可以检索 [`EXTRA_PERMISSION_GROUP_NAME`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#EXTRA_PERMISSION_GROUP_NAME) 的值。
    - 如果系统调用 `ACTION_VIEW_PERMISSION_USAGE_FOR_PERIOD`，您的应用可以检索 `EXTRA_PERMISSION_GROUP_NAME`、[`EXTRA_ATTRIBUTION_TAGS`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#EXTRA_ATTRIBUTION_TAGS)、[`EXTRA_START_TIME`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#EXTRA_START_TIME) 和 [`EXTRA_END_TIME`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#EXTRA_END_TIME) 的值。

根据您添加的 intent 过滤器，用户会在某些屏幕上看到应用的名称旁边有一个信息图标：

- 如果您添加包含 `VIEW_PERMISSION_USAGE` 操作的 intent 过滤器，用户会在系统设置中的应用权限页面上看到该图标。您可以将该操作应用于所有运行时权限。
- 如果您添加包含 `VIEW_PERMISSION_USAGE_FOR_PERIOD` 操作的 intent 过滤器，每当您的应用显示在“隐私信息中心”屏幕中，用户都会在应用的名称旁边看到该图标。

当用户选择该图标时，系统会启动应用的理由 activity。


## 指示标志

>**注意**：本节中提到的图标应该不需要更改应用程序的逻辑，只要您[遵循隐私设置最佳做法](https://developer.android.com/privacy/best-practices?hl=zh-cn)。

在搭载 Android 12 或更高版本的设备上，当应用使用麦克风或相机时，图标会出现在状态栏中。如果应用处于[沉浸模式](https://developer.android.com/training/system-ui/immersive?hl=zh-cn#immersive)，图标会出现在屏幕的右上角。用户可以打开“快捷设置”，并选择图标以查看哪些应用当前正在使用麦克风或摄像头。图 2 显示了包含图标的示例屏幕截图。

![右上角的圆角矩形，其中包含相机图标和麦克风图标](https://rd-wang.github.io/assets/img/android/permissions//mic-camera-indicators.svg)

**图 2.** 麦克风和摄像头指示标志，显示了最近的数据访问。

### 确定指示标志在屏幕上的位置

如果您的应用支持沉浸模式或全屏界面，指示标志可能会与应用界面短暂重叠。为协助应用界面适应这些指示标志，系统引入了 [`getPrivacyIndicatorBounds()`](https://developer.android.com/reference/android/view/WindowInsets.Builder?hl=zh-cn#setPrivacyIndicatorBounds\(android.graphics.Rect\)) 方法，如下方的代码段所示。利用此 API，您可以确定指示标志可能出现的边界。然后，您可能会决定以不同的布局安排屏幕界面。

[Kotlin](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#kotlin)

```kotlin
view.setOnApplyWindowInsetsListener { view, windowInsets ->
    val indicatorBounds = windowInsets.getPrivacyIndicatorBounds()
    // change your UI to avoid overlapping
    windowInsets
}
```

## 切换开关

>**注意**：本节中提到的开关应该不需要更改应用程序的逻辑，只要您[遵循隐私设置最佳做法](https://developer.android.com/privacy/best-practices?hl=zh-cn)。


在搭载 Android 12 或更高版本的[受支持设备](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#toggles-check-device-support)上，用户可以通过按一个切换开关选项，为设备上的所有应用启用和停用摄像头和麦克风使用权限。用户可以从[快捷设置](https://support.google.com/android/answer/9083864?hl=zh-cn)访问可切换的选项（如图 3 所示），也可以从系统设置中的“隐私设置”屏幕访问。

![快捷设置图块标有“摄像头使用权限”和“麦克风使用权限”](https://rd-wang.github.io/assets/img/android/permissions/mic-camera-toggles.svg)

**图 3.** “快捷设置”中的麦克风和摄像头切换开关。

摄像头和麦克风切换开关会影响设备上的所有应用：

- 当用户关闭摄像头访问权限后，您的应用会收到空白的摄像头画面。
- 当用户关闭麦克风使用权限后，您的应用会收到无声音频。此外，无论您是否声明 [`HIGH_SAMPLING_RATE_SENSORS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#HIGH_SAMPLING_RATE_SENSORS) 权限，[运动传感器的采样率都会受到限制](https://developer.android.com/guide/topics/sensors/sensors_overview?hl=zh-cn#sensors-rate-limiting)。
    
    **注意**：当用户拨打应急服务电话（如 911）时，系统会开启麦克风使用权限。此行为可保护用户安全。
    

当用户关闭摄像头或麦克风的使用权限，然后启动需要使用摄像头或麦克风信息的应用时，系统会提醒用户，设备范围的切换开关已关闭。

### 检查设备支持情况

如需检查设备是否支持麦克风和摄像头切换开关，请添加以下代码段中所示的逻辑：

[Kotlin](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#kotlin)
```kotlin
val sensorPrivacyManager = applicationContext
        .getSystemService(SensorPrivacyManager::class.java)
        as SensorPrivacyManager
val supportsMicrophoneToggle = sensorPrivacyManager
        .supportsSensorToggle(Sensors.MICROPHONE)
val supportsCameraToggle = sensorPrivacyManager
        .supportsSensorToggle(Sensors.CAMERA)
```
[Java](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#java)

```java
SensorPrivacyManager sensorPrivacyManager = getApplicationContext()
        .getSystemService(SensorPrivacyManager.class);
boolean supportsMicrophoneToggle = sensorPrivacyManager
        .supportsSensorToggle(Sensors.MICROPHONE);
boolean supportsCameraToggle = sensorPrivacyManager
        .supportsSensorToggle(Sensors.CAMERA);
```