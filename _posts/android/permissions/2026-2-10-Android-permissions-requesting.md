---
title: Android-权限-请求运行时权限
date: 2026-2-6 16:07:12 +0800
categories:
  - Android
  - Permissions
  - Request
tags:
  - Android
  - Permissions
  - Request
description: 每个 Android 应用都在访问权受限的沙盒中运行。如果您的应用需要使用自身沙盒以外的资源或信息，您可以声明运行时权限并设置一项权限请求来获取所需访问权限。
math: true
---
# 请求运行时权限

每个 Android 应用都在访问权受限的沙盒中运行。如果您的应用需要使用自身沙盒以外的资源或信息，您可以[声明运行时权限](https://developer.android.com/training/permissions/declaring?hl=zh-cn)并设置一项权限请求来获取所需访问权限。以下步骤是[权限使用工作流](https://developer.android.com/training/basics/permissions?hl=zh-cn#workflow)的一部分。

![点击观看引导视频](https://youtu.be/x38dYUm7tCY)

>**注意** ：有些权限旨在限制访问尤其敏感或与用户隐私没有直接关系的系统资源。[请求这些特殊权限的流程有所不同](https://developer.android.com/training/permissions/requesting-special?hl=zh-cn)。

如果您声明了任何[危险权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#dangerous_permissions)，并且您的应用安装在搭载 Android 6.0（API 级别 23）或更高版本的设备上，那么您必须按照本指南中的步骤在运行时请求这些危险权限。

如果您没有声明任何危险权限，或者您的应用安装在搭载 Android 5.1（API 级别 22）或更低版本的设备上，则系统会自动授予相应的权限，您无需完成本页剩下的任何步骤。

## 基本原则

在运行时请求权限的基本原则如下：

- 在用户开始与需要权限的功能进行交互时，根据上下文请求权限。
- 不要阻止用户。始终提供取消教学流程的选项，例如解释请求权限理由的流程。
- 如果用户拒绝或撤销某个功能所需的权限，请优雅地降低应用程序的运行级别，以便用户可以继续使用应用程序，例如禁用需要该权限的功能。
- 不要对系统行为做任何假设。例如，不要假设权限会出现在同一个权限组中。权限组的作用仅仅是帮助系统减少当应用程序请求密切相关的权限时向用户显示的系统对话框的数量。

## 请求权限的工作流

在应用中声明并请求运行时权限之前，请[评估您的应用是否需要这样做](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)。您无需声明任何权限就可以在您的应用中实现很多用例，例如拍照、暂停媒体播放和展示相关广告。

如果您确定您的应用需要声明和请求运行时权限，请完成以下步骤：

1. 在应用的清单文件中[**声明**应用可能需要请求的权限](https://developer.android.com/training/permissions/declaring?hl=zh-cn)。
2. **设计**应用的用户体验 (UX)时，应确保应用中的特定操作与特定的运行时权限相关联。让用户了解哪些操作可能需要他们授予应用访问用户隐私数据的权限。
3. [**等待**用户](https://developer.android.com/training/permissions/requesting?hl=zh-cn#principles)在您的应用中执行需要访问特定私有用户数据的任务或操作。届时，您的应用可以请求访问该数据所需的运行时权限。
4. [**检查**用户是否已授予](https://developer.android.com/training/permissions/requesting?hl=zh-cn#already-granted)应用所需的运行时权限。如果已授权，那么您的应用可以访问用户私人数据。如果没有，请继续执行下一步。
    
    每次执行需要该权限的操作时，您都必须检查自己是否具有该权限。
    
5. [**检查**您的应用是否应向用户显示理由](https://developer.android.com/training/permissions/requesting?hl=zh-cn#explain)，说明您的应用需要用户授予特定运行时权限的原因。如果系统确定您的应用不应显示理由，请继续直接执行下一步，无需显示界面元素。
    
    不过，如果系统确定您的应用应该显示一个理由，请在界面元素中向用户显示理由，明确说明您的应用试图访问哪些数据，以及应用获得运行时权限后可为用户提供哪些好处。用户确认理由后，请继续执行下一步。
    
6. [**请求**您的应用访问用户私人数据所需的](https://developer.android.com/training/permissions/requesting?hl=zh-cn#request-permission)运行时权限。系统会显示运行时权限提示，例如[权限概览页面](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#fig-runtime)上显示的提示。
    
7. **检查**用户的响应，他们可能会选择同意或拒绝授予运行时权限。
    
8. 如果用户向您的应用授予相应权限，您就可以**访问**用户私人数据。如果用户拒绝授予该权限，请[适当降低应用体验](https://developer.android.com/training/permissions/requesting?hl=zh-cn#handle-denial)，使应用在未获得受该权限保护的信息时也能向用户提供功能。
    

图 1 说明了与此过程相关的工作流和决策组：

![一张流程图，用于说明请求运行时权限的决策流程，从应用操作开始，检查是否已授予权限，检查是否需要理由，请求权限，以及处理用户的响应。](https://rd-wang.github.io/assets/img/android/permissions/workflow-runtime.svg){: width="100%" height="100%" }

**图 1.** 在 Android 上声明和请求运行时权限的工作流。

## 确定应用是否已获得权限

如需检查用户是否已向您的应用授予特定权限，请将该权限传入 [`ContextCompat.checkSelfPermission()`](https://developer.android.com/reference/androidx/core/content/ContextCompat?hl=zh-cn#checkSelfPermission\(android.content.Context,%20java.lang.String\)) 方法。根据您的应用是否具有相应权限，此方法会返回 [`PERMISSION_GRANTED`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#PERMISSION_GRANTED) 或 [`PERMISSION_DENIED`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#PERMISSION_DENIED)。

## 说明您的应用为何需要获取权限

系统在您调用 [`requestPermissions()`](https://developer.android.com/reference/androidx/core/app/ActivityCompat?hl=zh-cn#requestPermissions\(android.app.Activity,%20java.lang.String%5B%5D,%20int\)) 时显示的权限对话框将说明应用需要哪些权限，但不会解释为何需要这些权限。在某些情况下，用户可能会感到困惑。因此，最好在调用 `requestPermissions()` 之前向用户解释应用需要相应权限的原因。

研究表明，如果用户知道应用需要权限的原因（例如需要权限来支持应用的核心功能或投放广告），他们会更容易接受权限请求。因此，如果您只使用了某个权限组下的一小部分 API 调用，请明确列出您正在使用哪些权限以及原因。例如，如果您仅使用粗略位置信息，请在应用说明或应用的帮助文章中告知用户。

在特定条件下，让用户实时了解应用在访问敏感数据也是很有帮助的。例如，如果您要访问摄像头或麦克风，请考虑在应用中的某个位置或通知托盘（如果应用在后台运行）中使用通知图标告知用户，以免用户认为您在偷偷收集数据。

>**注意** ：从 Android 12（API 级别 31）开始，每当应用使用麦克风或摄像头时，[隐私指示标志](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#indicators)都会通知用户。

如果您需要请求权限才能使某项功能正常运行，但用户不清楚原因，则需要找到一种方法来解释您为什么需要敏感权限。

如果 `ContextCompat.checkSelfPermission()` 方法返回 `PERMISSION_DENIED`，请调用 [`shouldShowRequestPermissionRationale()`](https://developer.android.com/reference/androidx/core/app/ActivityCompat?hl=zh-cn#shouldShowRequestPermissionRationale\(android.app.Activity,%20java.lang.String\))。如果此方法返回 `true`，请向用户显示指导界面。在此界面中说明用户希望启用的功能为何需要特定权限。

此外，如果您的应用请求与位置信息、麦克风或相机相关的权限，请考虑[说明该应用需要访问这些信息的原因](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn)。

## 请求权限

用户查看指导界面后或者 `shouldShowRequestPermissionRationale()` 的返回值表明您不需要显示指导界面后，您可以请求权限。用户会看到一个系统权限对话框，他们可以选择是否授予您的应用特定权限。

为此，请使用 AndroidX 库中包含的 [`RequestPermission`](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts.RequestPermission?hl=zh-cn) 协定类，您可在其中[允许系统替您管理权限请求代码](https://developer.android.com/training/permissions/requesting?hl=zh-cn#allow-system-manage-request-code)。由于使用 `RequestPermission` 协定类可简化逻辑，因此建议您尽量使用这种解决方案。不过，如果需要，您也可以在权限请求过程中[自行管理请求代码](https://developer.android.com/training/permissions/requesting?hl=zh-cn#manage-request-code-yourself)，并将该请求代码添加到权限回调逻辑中。

### 允许系统管理权限请求代码

如需允许系统管理与权限请求相关联的请求代码，请在您模块的 `build.gradle` 文件中添加以下库的依赖项：

- [`androidx.activity`](https://developer.android.com/jetpack/androidx/releases/activity?hl=zh-cn#declaring_dependencies)，版本 1.2.0 或更高版本
- [`androidx.fragment`](https://developer.android.com/jetpack/androidx/releases/fragment?hl=zh-cn#declaring_dependencies)，版本 1.3.0 或更高版本

然后，您可以使用以下某个类：

- 如需请求一项权限，请使用 [`RequestPermission`](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts.RequestPermission?hl=zh-cn)。
- 如需同时请求多项权限，请使用 [`RequestMultiplePermissions`](https://developer.android.com/reference/androidx/activity/result/contract/ActivityResultContracts.RequestMultiplePermissions?hl=zh-cn)。

以下步骤显示了如何使用 `RequestPermission` 协定类。使用 `RequestMultiplePermissions` 协定类的流程基本与此相同。

1. 在 activity 或 fragment 的初始化逻辑中，将 [`ActivityResultCallback`](https://developer.android.com/reference/androidx/activity/result/ActivityResultCallback?hl=zh-cn) 的实现传递给 [`registerForActivityResult()`](https://developer.android.com/reference/androidx/activity/result/ActivityResultCaller?hl=zh-cn#registerForActivityResult\(androidx.activity.result.contract.ActivityResultContract%3CI,%20O%3E,%20androidx.activity.result.ActivityResultCallback%3CO%3E\)) 的调用。`ActivityResultCallback` 定义了你的应用如何处理用户对权限请求的响应。
    
    保留对 `registerForActivityResult()`（类型为 [`ActivityResultLauncher`](https://developer.android.com/reference/androidx/activity/result/ActivityResultLauncher?hl=zh-cn)）的返回值的引用。
    
2. 如需在必要时显示系统权限对话框，请对您在上一步中保存的 `ActivityResultLauncher` 实例调用 [`launch()`](https://developer.android.com/reference/androidx/activity/result/ActivityResultLauncher?hl=zh-cn#launch\(I\)) 方法。
    
    调用 `launch()` 之后，系统会显示系统权限对话框。当用户做出选择后，系统会异步调用您在上一步中定义的 `ActivityResultCallback` 实现。
    
    **注意**：您的应用无法自定义调用 launch() 时出现的对话框。为了向用户提供更多信息或上下文，请更改应用的 UI，以便用户更容易理解应用中的某个功能为何需要特定权限。例如，您可以更改启用该功能的按钮上的文本。 此外，系统权限对话框中的文本会引用与您请求的权限关联的[权限组](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#group)。此权限分组旨在方便系统使用，您的应用不应依赖于权限是否属于特定的权限组。
    

以下代码段展示了如何处理权限响应：

[Kotlin](https://developer.android.com/training/permissions/requesting?hl=zh-cn#kotlin)
```kotlin
when {
    ContextCompat.checkSelfPermission(
            CONTEXT,
            Manifest.permission.REQUESTED_PERMISSION
            ) == PackageManager.PERMISSION_GRANTED -> {
        // You can use the API that requires the permission.
        performAction(...)
    }
    ActivityCompat.shouldShowRequestPermissionRationale(
            this, Manifest.permission.REQUESTED_PERMISSION) -> {
        // In an educational UI, explain to the user why your app requires this
        // permission for a specific feature to behave as expected, and what
        // features are disabled if it's declined. In this UI, include a
        // "cancel" or "no thanks" button that lets the user continue
        // using your app without granting the permission.
        showInContextUI(...)
    }
    else -> {
        // You can directly ask for the permission.
        requestPermissions(CONTEXT,
                arrayOf(Manifest.permission.REQUESTED_PERMISSION),
                REQUEST_CODE)
    }
}
```
[Java](https://developer.android.com/training/permissions/requesting?hl=zh-cn#java)
``
```java
if (ContextCompat.checkSelfPermission(
        CONTEXT, Manifest.permission.REQUESTED_PERMISSION) ==
        PackageManager.PERMISSION_GRANTED) {
    // You can use the API that requires the permission.
    performAction(...);
} else if (ActivityCompat.shouldShowRequestPermissionRationale(
        this, Manifest.permission.REQUESTED_PERMISSION)) {
    // In an educational UI, explain to the user why your app requires this
    // permission for a specific feature to behave as expected, and what
    // features are disabled if it's declined. In this UI, include a
    // "cancel" or "no thanks" button that lets the user continue
    // using your app without granting the permission.
    showInContextUI(...);
} else {
    // You can directly ask for the permission.
    requestPermissions(CONTEXT,
            new String[] { Manifest.permission.REQUESTED_PERMISSION },
            REQUEST_CODE);
}
```
``
以下代码片段演示了检查权限以及在必要时向用户请求权限的推荐流程：

[Kotlin](https://developer.android.com/training/permissions/requesting?hl=zh-cn#kotlin)
```kotlin
when {
    ContextCompat.checkSelfPermission(
            CONTEXT,
            Manifest.permission.REQUESTED_PERMISSION
            ) == PackageManager.PERMISSION_GRANTED -> {
        // You can use the API that requires the permission.
    }
    ActivityCompat.shouldShowRequestPermissionRationale(
            this, Manifest.permission.REQUESTED_PERMISSION) -> {
        // In an educational UI, explain to the user why your app requires this
        // permission for a specific feature to behave as expected, and what
        // features are disabled if it's declined. In this UI, include a
        // "cancel" or "no thanks" button that lets the user continue
        // using your app without granting the permission.
        showInContextUI(...)
    }
    else -> {
        // You can directly ask for the permission.
        // The registered ActivityResultCallback gets the result of this request.
        requestPermissionLauncher.launch(
                Manifest.permission.REQUESTED_PERMISSION)
    }
}
```
[Java](https://developer.android.com/training/permissions/requesting?hl=zh-cn#java)

```java
if (ContextCompat.checkSelfPermission(
        CONTEXT, Manifest.permission.REQUESTED_PERMISSION) ==
        PackageManager.PERMISSION_GRANTED) {
    // You can use the API that requires the permission.
    performAction(...);
} else if (ActivityCompat.shouldShowRequestPermissionRationale(
        this, Manifest.permission.REQUESTED_PERMISSION)) {
    // In an educational UI, explain to the user why your app requires this
    // permission for a specific feature to behave as expected, and what
    // features are disabled if it's declined. In this UI, include a
    // "cancel" or "no thanks" button that lets the user continue
    // using your app without granting the permission.
    showInContextUI(...);
} else {
    // You can directly ask for the permission.
    // The registered ActivityResultCallback gets the result of this request.
    requestPermissionLauncher.launch(
            Manifest.permission.REQUESTED_PERMISSION);
}
```

### 自行管理权限请求代码

作为[允许系统管理权限请求代码](https://developer.android.com/training/permissions/requesting?hl=zh-cn#allow-system-manage-request-code)的替代方法，您可以自行管理权限请求代码。为此，请在对 [`requestPermissions()`](https://developer.android.com/reference/androidx/core/app/ActivityCompat?hl=zh-cn#requestPermissions\(android.app.Activity,%20java.lang.String%5B%5D,%20int\)) 的调用中添加请求代码。

> **注意**：应用无法自定义调用 `requestPermissions()` 时显示的对话框。系统权限对话框中的文本会提及[权限组](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#perm-groups)，但此权限分组是为了让系统易于使用。您的应用不得依赖于特定权限组之内或之外的权限。

以下代码段演示了如何使用请求代码来请求权限：

[Kotlin](https://developer.android.com/training/permissions/requesting?hl=zh-cn#kotlin)
```kotlin
when {
    ContextCompat.checkSelfPermission(
            CONTEXT,
            Manifest.permission.REQUESTED_PERMISSION
            ) == PackageManager.PERMISSION_GRANTED -> {
        // You can use the API that requires the permission.
        performAction(...)
    }
    ActivityCompat.shouldShowRequestPermissionRationale(
            this, Manifest.permission.REQUESTED_PERMISSION) -> {
        // In an educational UI, explain to the user why your app requires this
        // permission for a specific feature to behave as expected, and what
        // features are disabled if it's declined. In this UI, include a
        // "cancel" or "no thanks" button that lets the user continue
        // using your app without granting the permission.
        showInContextUI(...)
    }
    else -> {
        // You can directly ask for the permission.
        requestPermissions(CONTEXT,
                arrayOf(Manifest.permission.REQUESTED_PERMISSION),
                REQUEST_CODE)
    }
}
```
[Java](https://developer.android.com/training/permissions/requesting?hl=zh-cn#java)

```java
if (ContextCompat.checkSelfPermission(
        CONTEXT, Manifest.permission.REQUESTED_PERMISSION) ==
        PackageManager.PERMISSION_GRANTED) {
    // You can use the API that requires the permission.
    performAction(...);
} else if (ActivityCompat.shouldShowRequestPermissionRationale(
        this, Manifest.permission.REQUESTED_PERMISSION)) {
    // In an educational UI, explain to the user why your app requires this
    // permission for a specific feature to behave as expected, and what
    // features are disabled if it's declined. In this UI, include a
    // "cancel" or "no thanks" button that lets the user continue
    // using your app without granting the permission.
    showInContextUI(...);
} else {
    // You can directly ask for the permission.
    requestPermissions(CONTEXT,
            new String[] { Manifest.permission.REQUESTED_PERMISSION },
            REQUEST_CODE);
}
```

当用户响应系统权限对话框后，系统就会调用应用的 [`onRequestPermissionsResult()`](https://developer.android.com/reference/androidx/core/app/ActivityCompat.OnRequestPermissionsResultCallback?hl=zh-cn#onRequestPermissionsResult\(int,%20java.lang.String%5B%5D,%20int%5B%5D\)) 实现。系统会将用户对权限对话框的响应以及您定义的请求代码传递给系统，如下面的代码片段所示：

[Kotlin](https://developer.android.com/training/permissions/requesting?hl=zh-cn#kotlin)
```kotlin
override fun onRequestPermissionsResult(requestCode: Int,
        permissions: Array<String>, grantResults: IntArray) {
    when (requestCode) {
        PERMISSION_REQUEST_CODE -> {
            // If request is cancelled, the result arrays are empty.
            if ((grantResults.isNotEmpty() &&
                    grantResults[0] == PackageManager.PERMISSION_GRANTED)) {
                // Permission is granted. Continue the action or workflow
                // in your app.
            } else {
                // Explain to the user that the feature is unavailable because
                // the feature requires a permission that the user has denied.
                // At the same time, respect the user's decision. Don't link to
                // system settings in an effort to convince the user to change
                // their decision.
            }
            return
        }

        // Add other 'when' lines to check for other
        // permissions this app might request>.
        else - {
            // Ignore all other requests.
        }
    }
}
```
[Java](https://developer.android.com/training/permissions/requesting?hl=zh-cn#java)

```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions,
        int[] grantResults) {
    switch (requestCode) {
        case PERMISSION_REQUEST_CODE:
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0 &&  
                    grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permission is granted. Continue the action or workflow
                // in your app.
            }  else {
                // Explain to the user that the feature is unavailable because
                // the feature requires a permission that the user has denied.
                // At the same time, respect the user's decision. Don't link to
                // system settings in an effort to convince the user to change
                // their decision.
            }
            return;
        }
        // Other 'case' lines to check for other
        // permissions this app might request.
    }
}
```

### 请求位置信息权限

请求位置信息权限时，请遵循与请求任何其他[运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn)相同的最佳实践。请求位置权限时的一个重要区别在于，系统中包含与位置相关的多项权限。具体请求哪项权限以及请求相关权限的方式取决于应用用例的位置信息要求。

#### 前台位置信息

如果应用的某项功能仅分享或接收一次位置信息，或者只在特定的一段时间内分享或接收位置信息，则该功能需要前台位置信息访问权限。以下是此类情况的一些示例：

- 在导航应用中，某项功能可让用户查询精细导航路线。
- 在即时通讯应用中，某项功能可让用户与其他用户分享自己当前的位置信息。

如果应用的功能在下列某种情况下访问设备当前的位置信息，系统就会认为应用需要使用前台位置信息：

- 属于应用的某个 activity 可见。
- 应用的某个前台服务正在运行中。当有前台服务在运行时，系统会显示一条常驻通知来提醒用户注意。当应用被置于后台时（例如当用户按设备上的**主屏幕**按钮或关闭设备的显示屏时），其位置信息访问权限会得到保留。
    
    在 Android 10（API 级别 29）及更高版本中，您必须声明[前台服务类型](https://developer.android.com/guide/topics/manifest/service-element?hl=zh-cn#foregroundservicetype) `location`，如以下代码段所示。在早期版本的 Android 中，建议您声明此前台服务类型。
    
 ```xml
    <!-- Recommended for Android 9 (API level 28) and lower. -->
    <!-- Required for Android 10 (API level 29) and higher. -->
    <service
	    android:name=".LocationForegroundService"
	    android:exported="false"
	    android:foregroundServiceType="location" />
 ```
    

当应用请求 [`ACCESS_COARSE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_COARSE_LOCATION) 权限或 [`ACCESS_FINE_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_FINE_LOCATION) 权限时（如以下代码段所示），就是在声明需要获取前台位置信息：

```xml
<manifest ... >

    <!-- 只要应用需要位置信息，就必须声明此权限（粗略定位） -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <!-- 如果应用需要精确定位（GPS），再额外声明此权限 -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

</manifest>
```

#### 后台位置信息

如果应用中的某项功能会不断与其他用户分享位置信息或使用 [Geofencing API](https://developer.android.com/training/location/geofencing?hl=zh-cn)，则该应用需要后台位置信息访问权限。以下是此类情况的几个示例：

- 在家庭位置信息分享应用中，某项功能可让用户与家庭成员持续分享位置信息。
- 在 IoT 应用中，某项功能可让用户配置自己的家居设备，使其在用户离家时关机并在用户回家时重新开机。

除了[前台位置信息](https://developer.android.com/training/permissions/requesting?hl=zh-cn#foreground-location)部分所述的情况之外，如果应用在任何其他情况下访问设备的当前位置信息，系统就会认为应用需要使用后台位置信息。后台位置信息精确度与[前台位置信息精确度](https://developer.android.com/training/location/permissions?hl=zh-cn#accuracy)相同，具体取决于应用声明的位置信息权限。

在 Android 10（API 级别 29）及更高版本中，您必须在应用的清单中声明 [`ACCESS_BACKGROUND_LOCATION`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#ACCESS_BACKGROUND_LOCATION) 权限，以便请求在运行时于后台访问位置信息。在较低版本的 Android 系统中，当应用获得前台位置信息访问权限时，也会自动获得后台位置信息访问权限。

```xml
<manifest ... >
  <!-- Required only when requesting background location access on
       Android 10 (API level 29) and higher. -->
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

> **注意**： Google Play 商店设置了有关设备位置信息的[位置信息政策](https://support.google.com/googleplay/android-developer/answer/9799150?hl=zh-cn)，限制应用仅在实现核心功能所必需的情形下且在满足相关政策要求后才能请求后台位置信息访问权限。

## 处理权限请求遭拒情况

如果用户拒绝了权限请求，应用必须帮助用户了解拒绝授予权限的影响。具体而言，应用必须让用户知道因缺少权限而无法使用哪些功能。在处理这种情况时，请牢记以下最佳做法：

- **引导用户的注意力**。在应用界面中突出显示因为应用没有获得必要的权限而受限的功能所在的具体部分。您可以采取的行动（示例）包括：
    
    - 在原本用于显示该功能的结果或数据的位置显示一条消息。
    - 显示一个包含错误图标并带有相应错误颜色的不同**按钮**。
- **内容要具体**。显示的消息不要空泛，而要指出因为应用没有获得必要的权限而无法使用的具体功能。
    
- **不要阻止界面显示。** 换言之，不要显示全屏警告消息，让用户根本无法继续使用您的应用。
    

**提示**： 我们建议您的应用尽可能提供最佳的用户体验，即使在权限请求遭拒后也是如此。例如，即使麦克风使用权限请求遭拒，您仍然必须确保用户能够完全使用文本功能。

与此同时，您的应用还必须尊重用户拒绝授予权限的决定。从 Android 11（API 级别 30）开始，在应用安装到设备上后，如果用户在使用过程中多次针对某项特定的权限点按**拒绝**，那么在您的应用再次请求该权限时，用户将不会看到系统权限对话框。该操作表示用户希望“不再询问”。在之前的版本中，除非用户先前已选中**不再询问**复选框或选项，否则每当您的应用请求权限时，用户都会看到系统权限对话框。

如果用户多次拒绝权限请求，则视为永久拒绝。因此，务必仅在用户需要访问特定功能时才提示其授予权限；否则，您可能会无意中失去重新请求权限的能力。

在某些情况下，系统可能会自动拒绝权限，而无需用户执行任何操作（系统也可能会自动授予权限）。请千万不要对系统的自动行为做出任何假设。每当应用需要使用的功能需要权限时，请检查应用是否仍被授予该权限。

如需在请求应用权限时提供最佳用户体验，另请参阅[应用权限最佳实践](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn)。

### 在测试和调试时检查遭拒状态

如需确定应用是否已被用户永久拒绝授予权限（用于调试和测试），请使用以下命令：

```bash
adb shell dumpsys package PACKAGE_NAME
```

其中，PACKAGE_NAME 是要检查的软件包的名称。

该命令的输出包含如下部分：

```bash
...
runtime permissions:
  android.permission.POST_NOTIFICATIONS: granted=false, flags=[ USER_SENSITIVE_WHEN_GRANTED|USER_SENSITIVE_WHEN_DENIED]
  android.permission.ACCESS_FINE_LOCATION: granted=false, flags=[ USER_SET|USER_FIXED|USER_SENSITIVE_WHEN_GRANTED|USER_SENSITIVE_WHEN_DENIED]
  android.permission.BLUETOOTH_CONNECT: granted=false, flags=[ USER_SENSITIVE_WHEN_GRANTED|USER_SENSITIVE_WHEN_DENIED]
...
```
被用户拒绝过一次的权限由 `USER_SET` 标记。由于用户选择了两次**拒绝**而被永久拒绝的权限由 `USER_FIXED` 标记。

为确保测试人员在测试期间看到请求对话框，请在完成应用调试后重置这些标志。为此，请使用以下命令：

```bash
adb shell pm clear-permission-flags PACKAGE_NAME PERMISSION_NAME user-set user-fixed
```

PERMISSION_NAME 是您要重置的权限的名称。

`PERMISSION_NAME` 是您要重置的权限的名称。

如需查看 Android 应用权限的完整列表，请访问[权限 API 参考文档页面](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#constants_1)。

## 单次授权

从 Android 11（API 级别 30）开始，每当您的应用请求与位置信息、麦克风或摄像头相关的权限时，面向用户显示的权限对话框都会包含一个名为**仅限这一次**的选项，如图 2 所示。如果用户在对话框中选择此选项，系统会向应用授予临时的单次授权。

![对话框里有三个按钮，名为“仅限这一次”的选项是其中的第二个按钮。](https://rd-wang.github.io/assets/img/android/permissions/one-time-prompt.svg){: width="100%" height="100%" }

**图 2.** 应用请求单次授权时显示的系统对话框。

然后，应用可以在一段时间内访问相关数据，具体时间取决于应用的行为和用户的操作：

- 当应用的 activity 可见时，应用可以访问相关数据。
- 如果用户将应用转为后台运行，应用可以在短时间内继续访问相关数据。
- 如果您在 activity 可见时启动了一项前台服务，并且用户随后将您的应用转到后台，那么您的应用可以继续访问相关数据，直到该前台服务停止。

### 应用进程在权限被撤消时终止

如果用户撤消单次授权（例如在系统设置中撤消），无论您是否启动了前台服务，应用都无法访问相关数据。与任何权限一样，如果用户撤消了应用的单次授权，应用进程就会终止。

当用户下次打开应用并且应用中的某项功能请求访问位置信息、麦克风或摄像头时，系统会再次提示用户授予权限。

> **注意**： 如果应用在[请求运行时权限](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn)时已遵循最佳实践，那么您无需在应用中添加或更改任何逻辑即可支持单次授权。

## 重置未使用的权限

Android 提供了多种方法来将未使用的运行时权限重置为默认的拒绝状态：

- API，可主动[取消应用对未使用的运行时权限的访问权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn#remove-app-access)。
- 系统机制，可自动[重置未使用的应用的权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn#auto-reset-permissions-unused-apps)。

### 取消应用访问权限

在 Android 13（API 级别 33）及更高版本中，您可以取消应用对不再需要的运行时权限的访问权限。更新应用时，请执行此步骤，以便用户更有可能了解您的应用继续请求特定权限的原因。这些信息有助于建立用户对应用的信任。

如需取消对运行时权限的访问权限，请将该权限的名称传递到 [`revokeSelfPermissionOnKill()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#revokeSelfPermissionOnKill\(java.lang.String\))。如需同时取消对一组运行时权限的访问权限，请将这组权限名称传递到 [`revokeSelfPermissionsOnKill()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#revokeSelfPermissionsOnKill\(java.util.Collection%3Cjava.lang.String%3E\))。权限取消过程是异步执行的，会终止与应用 UID 关联的所有进程。

> **注意**： 为了让系统设置显示您的应用不会访问特定[权限组](https://developer.android.com/reference/android/Manifest.permission_group?hl=zh-cn)中的数据，您必须取消对该权限组中的**所有**权限的访问权限。在这种情况下，调用 `revokeSelfPermissionsOnKill()` 并传入权限组内的多项权限会很有帮助。

为了让系统取消应用对权限的访问权限，必须终止与应用关联的所有进程。当您调用该 API 时，系统会确定何时可以安全终止这些进程。通常，系统会等待应用有较长时间在后台运行，而不是在前台运行时。

要通知用户您的应用不再需要对特定运行时权限的访问权限，请在用户下次启动应用时显示一个对话框。此对话框可以包含权限列表。

### 自动重置未使用的应用的权限

如果您的应用以 Android 11（API 级别 30）或更高版本为目标平台并且数月未使用，系统会通过自动重置用户已授予应用的运行时敏感权限来保护用户数据。如需了解详情，请参阅有关[应用休眠](https://developer.android.com/topic/performance/app-hibernation?hl=zh-cn)的指南。

## 在必要时请求成为默认处理程序

有些应用依赖于访问与通话记录和短信有关的敏感用户信息。如果您想请求特定于通话记录和短信的权限，并将应用发布到 Play 商店，您必须在请求这些运行时权限之前，提示用户将应用设置为核心系统功能的默认处理程序。

如需详细了解默认处理程序，包括有关如何向用户显示默认处理程序提示的指南，请[参阅有关仅在默认处理程序中使用的权限的指南](https://developer.android.com/guide/topics/permissions/default-handlers?hl=zh-cn)。

## 授予所有运行时权限以进行测试

如需在您在模拟器或测试设备上安装应用时自动授予所有运行时权限，请为 `adb shell install` 命令使用 `-g` 选项，如以下代码段所示：

```bash
adb shell install -g PATH_TO_APK_FILE
```

## 其他资源

如需详细了解权限，请阅读以下文章：

- [权限概览](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn)
- [应用权限最佳做法](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn)

如需详细了解如何请求权限，请参阅[权限示例](https://github.com/android/platform-samples/tree/main/samples/privacy/permissions)。

您还可以完成这个[演示隐私权最佳实践的 Codelab](https://developer.android.com/codelabs/android-privacy-codelab?hl=zh-cn)。