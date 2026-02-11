---
title: Android-权限-请求特殊权限
date: 2026-2-11 14:11:06 +0800
categories:
  - Android
  - Permissions
  - Request
tags:
  - Android
  - Permissions
  - Request
description: 特殊权限旨在限制访问尤其敏感或与用户隐私没有直接关系的系统资源。
math: true
---
# 请求特殊权限

特殊权限旨在限制访问尤其敏感或与用户隐私没有直接关系的系统资源。这些权限不同于[安装时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#install-time)和[运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)。

一些特殊权限示例：

- 设定精确的闹钟。
- 在其他应用前方显示和绘图。
- 访问所有存储数据。

声明特殊权限的应用会显示在系统设置中的**特殊应用权限**页面内（图 1）。如需向应用授予特殊权限，用户必须转到此页面：**设置 > 应用 > 特殊应用权限**。

![Android 系统设置的“特殊应用权限”界面，其中显示了应用列表及其特殊权限状态。](https://rd-wang.github.io/assets/img/android/permissions/special-app-access.svg)

**图 1.** 系统设置中的**特殊应用权限**界面。

> **注意**： 仅在特定用例中使用特殊权限。在应用中添加这些功能可能会带来政策方面的影响。


## 工作流程

如需请求特殊权限，请执行以下操作：

1. 在应用的清单文件中，[声明应用可能需要请求的特殊权限](https://developer.android.com/training/permissions/declaring?hl=zh-cn)。
2. 设计应用的用户体验 (UX)，使应用中的特定操作与特定的特殊权限相关联。告知用户哪些操作可能会要求他们向您的应用授予访问其私人数据的权限。
3. [等待用户调用](https://developer.android.com/training/permissions/requesting?hl=zh-cn#principles)应用中需要访问特定用户私人数据的任务或操作。届时，您的应用可以请求获得访问相应数据所需的特殊权限。
4. 检查用户是否已授予您的应用所需的特殊权限。为此，请使用每项权限的[自定义检查函数](https://developer.android.com/training/permissions/requesting-special?hl=zh-cn#check-method)。如果已获得授权，那么您的应用可以访问用户私人数据。如果没有，请继续执行下一步。注意：每次执行需要该权限的操作时，您的应用都必须检查是否拥有该权限。
5. 在界面元素中向用户显示理由，其中要清楚地解释您的应用在尝试访问哪些数据，以及用户为您的应用授予特殊权限后可获得哪些好处。此外，由于应用会将用户转到系统设置部分以便其授予权限，因此还要添加简要说明，解释用户如何在其中授予权限。理由界面应为用户提供明确的选项，让用户可以选择不授予相应权限。用户确认理由后，请继续执行下一步。
6. 请求您的应用访问用户私人数据所需的特殊权限。这可能涉及通过 intent 转到系统设置中的相应页面，以便用户授予权限。与[运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)不同，系统不会提供权限对话框。
7. 在 `onResume()` 方法中检查用户的响应，他们可能会选择同意或拒绝授予特殊权限。
8. 如果用户向您的应用授予权限，您就可以访问用户私人数据。如果用户拒绝授予该权限，请[适当降低应用体验](https://developer.android.com/training/permissions/requesting?hl=zh-cn#handle-denial)，使应用在未获得受该权限保护的信息时也能向用户提供功能。

![该图展示了在 Android 上声明和请求特殊权限的工作流，从清单声明到用户理由、系统设置重定向，再到处理用户决定。](https://rd-wang.github.io/assets/img/android/permissions/workflow-special.svg?hl=zh-cn)

**图 2.** 在 Android 上声明和请求特殊权限的工作流程。

## 请求特殊权限

与[运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)不同，用户必须从系统设置中的**特殊应用权限**页面授予特殊权限。应用可以使用 intent 将用户转到该页面，这会暂停应用，并启动相应的设置页面，以便用户授予指定的特殊权限。用户返回到应用后，应用可以在 `onResume()` 函数中检查是否已获得相应权限。

以下示例代码展示了如何请求用户授予 [`SCHEDULE_EXACT_ALARMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SCHEDULE_EXACT_ALARM) 特殊权限：

```kotlin
val alarmManager = getSystemService<AlarmManager>()!!
when {
   // if permission is granted, proceed with scheduling exact alarms…
   alarmManager.canScheduleExactAlarms() -> {
       alarmManager.setExact(...)
   }
   else -> {
       // ask users to grant the permission in the corresponding settings page
       startActivity(Intent(ACTION_REQUEST_SCHEDULE_EXACT_ALARM))
   }
}
```

在 `onResume()` 中检查权限和处理用户决定的示例代码：

```kotlin
override fun onResume() {
    // ...

    if (alarmManager.canScheduleExactAlarms()) {
        // proceed with the action (setting exact alarms)
        alarmManager.setExact(...)
    }
    else {
        // permission not yet approved. Display user notice and gracefully
        // degrade your app experience.
        alarmManager.setWindow(...)
    }
}
```

## 请求特殊权限的提示

以下部分提供了申请特殊权限时的注意事项和提示。

### 每项权限都有自己的检查方法

特殊权限的运作方式与[运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn#request-permission)不同。因此，请参阅[权限 API 参考文档页面](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn)，并针对每项特殊权限使用自定义权限检查函数。例如，使用 [`AlarmManager#canScheduleExactAlarms()`](https://developer.android.com/reference/android/app/AlarmManager?hl=zh-cn#canScheduleExactAlarms\(\)) 检查是否有 [`SCHEDULE_EXACT_ALARMS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SCHEDULE_EXACT_ALARM) 权限，以及使用 [`Environment#isExternalStorageManager()`](https://developer.android.com/reference/android/os/Environment?hl=zh-cn#isExternalStorageManager\(\)) 检查是否有 [`MANAGE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#MANAGE_EXTERNAL_STORAGE) 权限。

### 在上下文中请求

与运行时权限类似，应用应等到用户请求执行需要特殊权限的特定操作时，才在上下文中请求相应权限。例如，等到用户安排在特定时间发送电子邮件时，才请求 `SCHEDULE_EXACT_ALARMS` 权限。

### 解释相应请求

先提供理由，然后再转到系统设置。由于用户会暂时离开应用以授予特殊权限，因此在启动 intent 以转到系统设置中的**特殊应用权限**页面之前，要先显示应用内界面。此界面应清楚地解释应用为什么需要相应权限，以及用户应如何在设置页面中授予该权限。