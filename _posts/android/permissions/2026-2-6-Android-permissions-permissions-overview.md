---
title: Android-权限-Android 中的权限
date: 2026-2-6 16:08:40 +0800
categories:
  - Android
  - Permissions
  - Overview
tags:
  - Android
  - Permissions
  - Overview
description: 介绍 Android 权限的工作原理，包括使用权限的概要工作流、对不同类型权限的说明，以及在应用中使用权限的一些最佳实践。
math: true
---
应用权限有助于保护对以下数据和操作的访问/执行权限，从而为保护用户隐私提供支持：

- **受限数据**，例如系统状态和用户的联系信息
- **受限操作**，例如连接到已配对的设备并录制音频

本页将概要介绍 Android 权限的工作原理，包括使用权限的概要工作流、对不同类型权限的说明，以及在应用中使用权限的一些最佳实践。其他页面将介绍如何[最大限度减少应用的权限请求](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)、[声明权限](https://developer.android.com/training/permissions/declaring?hl=zh-cn)、[请求运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn)，以及[限制其他应用与应用组件交互的方式](https://developer.android.com/training/permissions/restrict-interactions?hl=zh-cn)。

如需查看 Android 应用权限的完整列表，请访问[权限 API 参考文档页面](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn)。

如需查看演示权限工作流的一些示例应用，请访问 GitHub 上的 [Android 权限示例仓库](https://github.com/android/platform-samples/tree/main/samples/privacy/permissions)。

如需查看官方视频介绍，请查看![Permissions Video](https://youtu.be/zCAx4WZ98rs)。
## 使用权限的工作流

如果您的应用提供的功能可能需要访问受限数据或执行受限操作，请确定您是否[无需声明权限](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)即可获取相关信息或执行相关操作。您可以在您的应用中实现很多场景（例如拍照、暂停媒体播放和展示相关广告），而无需声明任何权限。

如果您确定您的应用必须访问受限数据或执行受限操作才能实现某个使用场景，请声明相应的权限。有些权限是用户安装应用时自动授予的权限，称为[安装时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#install-time)。其他权限则需要应用在运行时进一步请求权限，此类权限称为[运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)。

图 1 显示了使用应用权限的工作流：

![图 1. 在 Android 中使用权限的概要工作流。](https://rd-wang.github.io/assets/img/android/permissions/workflow-overview.svg){: width="100%" height="100%" }



## 权限类型

Android 将权限分为不同的类型，包括安装时权限、运行时权限和特殊权限。每种权限类型都指明了当系统授予应用该权限后，应用可以访问的受限数据范围以及应用可以执行的受限操作范围。每项权限的保护级别取决于其类型，显示在[权限 API 参考文档](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn)页面上。

### 安装时权限


安装时权限允许您的应用有限地访问受限数据，或者允许您的应用执行对系统或其他应用影响最小的受限操作。当您在应用中声明安装时权限时，应用商店会在用户查看应用详情页面时向其显示安装时权限通知，如图 2 所示。当用户安装您的应用时，系统会自动授予您的应用相应的权限。

Android 包含几种安装时权限的子类型，包括普通权限和签名权限。

![应用商店中显示的应用安装时权限列表。](https://rd-wang.github.io/assets/img/android/permissions/install-time.svg){: width="100%" height="100%" }

**图 2.** 某应用商店中显示的某个应用的安装时权限列表。
#### 普通权限

此类权限允许访问超出应用沙盒的数据和执行超出应用沙盒的操作，但对用户隐私和其他应用的运行构成的风险很小。

系统会为一般权限分配 `normal` 保护级别。

#### 签名权限

只有当应用与定义权限的应用或 OS 使用相同的证书签名时，系统才会授予该应用程序签名权限。

实现特权服务（如自动填充或 VPN 服务）的应用也需要使用签名权限。这些应用需要服务绑定签名权限，以便只有系统可以绑定到服务。

> **注意**：有些签名权限不适合第三方应用使用。

系统会为签名权限分配 `signature` 保护级别。

### 运行时权限

运行时权限也称为危险权限，此类权限授予应用对受限数据的额外访问权限，或允许应用执行对系统和其他应用具有更严重影响的受限操作。因此，您需要先在应用中[请求运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn)，然后才能访问受限数据或执行受限操作。不要想当然地认为这些权限之前已被授予——每次访问前都要检查这些权限，如有必要，请先申请这些权限。

当应用请求运行时权限时，系统会显示运行时权限提示，如图 3 所示。

![当应用请求运行时权限时显示的系统权限提示](https://rd-wang.github.io/assets/img/android/permissions/runtime.svg){: width="100%" height="100%" }

**图 3.** 当应用请求运行时权限时显示的系统权限提示。

许多运行时权限会访问用户私人数据，这是一种特殊的受限数据，其中包括可能比较敏感的信息。例如，位置信息和联系信息就属于用户私人数据。

麦克风和摄像头可以访问极其敏感的信息。因此，系统会帮助您[说明应用获取这类信息的原因](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn)。

系统会为运行时权限分配 `dangerous` 保护级别。

### 特殊权限

特殊权限与特定的应用操作相对应。只有平台和原始设备制造商 (OEM) 可以定义特殊权限。此外，平台和 OEM 厂商通常会在想要保护对某些特别强大的操作（例如在其他应用程序上绘制图形）的访问时，定义特殊权限。

系统设置中的**特殊应用访问权限**页面包含一组用户可切换的操作。其中的许多操作都是以特殊权限的形式实现的。

详细了解如何[请求特殊权限](https://developer.android.com/training/permissions/requesting-special?hl=zh-cn)。

系统会为特殊权限分配 `appop` 保护级别。

### 权限组

权限可以属于[权限组](https://developer.android.com/guide/topics/manifest/permission-group-element.html?hl=zh-cn)。 权限组由一组逻辑相关的权限组成。例如，发送和接收短信的权限可能属于同一组，因为它们都与应用程序和短信的交互有关。

权限组的作用是在应用请求密切相关的多个权限时，最大限度地减少向用户显示的系统对话框的数量。当系统提示用户授予应用权限时，属于同一组的权限会在同一个界面中显示。 但是，权限可能会在不另行通知的情况下更改组，因此不要假设某个特定权限与其他任何权限分组在一起。

## 最佳实践

应用权限建立在[系统安全功能](https://source.android.com/security/features?hl=zh-cn)之上，并帮助 Android 支持以下与用户隐私相关的目标：

- **控制**：用户可以控制他们与应用分享的数据。
- **透明度**：用户了解应用使用了哪些数据以及应用为何访问相关数据。
- **数据最小化**：应用仅访问和使用用户调用的特定任务或操作所需的数据。

本部分将介绍一组在应用中有效使用权限的核心最佳实践。如需详细了解如何在 Android 中使用权限，请访问[应用权限最佳实践](https://developer.android.com/training/permissions/usage-notes?hl=zh-cn)页面。

### 请求最少数量的权限

当用户在应用中请求执行特定操作时，应用应当只请求完成该操作所需的权限。根据您使用权限的方式，您可以[通过其他方式实现应用的用例](https://developer.android.com/training/permissions/evaluating?hl=zh-cn)，而无需依赖于访问敏感信息。

### 将运行时权限与特定操作相关联

尽可能往后推迟到在应用的用例流程中请求权限。例如，如果应用允许用户向他人发送语音消息，请等到用户已导航到消息屏幕并已按下**发送语音消息**按钮后再请求权限。待用户按下该按钮后，应用再请求麦克风使用权限。

### 考虑应用的依赖项

添加某个库时，您也会继承它的权限要求。请注意每个依赖项所需的权限以及这些权限的用途。

### 公开透明

请求权限时，请清晰说明您要访问的内容、访问原因以及权限遭拒时哪些功能会受到影响，以便用户可以做出明智的决策。

### 以显式方式访问系统

当您访问敏感数据或硬件（例如摄像头或麦克风）时，如果系统尚未[提供这些指示标志](https://developer.android.com/training/permissions/explaining-access?hl=zh-cn#indicators)，请在应用中持续提供指示。此提醒可帮助用户确切了解应用何时会访问受限数据或执行受限操作。

## 系统组件中的权限

权限不仅仅用于请求系统功能。应用的系统组件可以限制哪些其他应用可以与您的应用交互，如介绍如何[限制与其他应用的交互](https://developer.android.com/training/permissions/restrict-interactions?hl=zh-cn)的页面中所述。