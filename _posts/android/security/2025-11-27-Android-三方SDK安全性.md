---
title: Android-三方SDK安全性
date: 2025-11-27 15:21:17 +0800
categories:
  - Android
  - Security
  - Third-party-SDK
tags:
  - Android
  - Security
  - Third-party-SDK
description: 
math: true
---

# 用户安全和 SDK 简介

作为**应用开发者**，您需要保护用户并确保应用能够安全稳定地运行，防范任何安全漏洞，包括可能由您所用的软件开发套件 (SDK) 引入的安全漏洞。

作为 **SDK 提供方**，您肯定不希望自己的 SDK 导致应用/游戏开发者违反 Google Play 开发者政策，进而使他们遭受业务中断并面临 Google Play 采取的违规处置措施。

无论您是使用 SDK 的应用开发者，还是 SDK 提供方，都有必要详细了解一下用户安全最佳实践。

## 针对应用开发者的说明

- 在将某个 SDK 集成到您的应用内之前，[确保您知道](https://medium.com/androiddevelopers/getting-to-know-the-behaviors-of-your-sdk-dependencies-f3dfed07a311)该 SDK 会使用什么权限、收集什么数据以及原因是什么。将这类信息添加到您的[数据安全表单](https://support.google.com/googleplay/android-developer/answer/10787469?hl=zh-cn)中。请注意，作为应用开发者，您必须对该 SDK 的数据收集行为负责，即使您并不使用该 SDK 的特定功能，也仍要负责。
- 查看所有[Google Play 开发者政策](https://play.google.com/about/developer-content-policy/?hl=zh-cn)，了解何时可以延长收集的用户数据的使用期限，以及何时不可以延长。例如，对于设备位置信息的使用，您必须通过[醒目披露声明和征求用户同意的要求](https://support.google.com/googleplay/android-developer/answer/11150561?hl=zh-cn)，告知最终用户您是否会与第三方/SDK 分享此类数据。
- 及时知晓 [Google Play 政策更新](https://support.google.com/googleplay/android-developer/answer/9934569?ref_topic=9877065&hl=zh-cn)，以确保您的应用内集成的 SDK 不会导致您的应用违反 Google Play 政策，例如[设备和网络滥用政策](https://support.google.com/googleplay/android-developer/answer/9888379?hl=zh-cn)、[广告政策](https://support.google.com/googleplay/android-developer/answer/9857753?ref_topic=9857752&hl=zh-cn)以及[用户数据政策中与永久标识符有关的规定](https://support.google.com/googleplay/android-developer/answer/10144311?hl=zh-cn)。
- 不出售个人信息以及敏感的用户信息。
- 如果应用的违规行为是因 SDK 所致，而您也因此收到违规处置通知，请参阅[有关如何在违反政策后重新提交应用的说明](https://support.google.com/googleplay/android-developer/answer/2477981?hl=zh-cn#resubmit)。
- 请访问 [Google Play SDK 索引](https://play.google.com/sdks?hl=zh-cn)，了解哪些 SDK 已在 Google Play 管理中心注册、这些 SDK 使用哪些 Android 权限等。

## 对于 SDK 提供方

- 了解 [Google Play 开发者政策](https://play.google.com/about/developer-content-policy/?hl=zh-cn)。
- 及时知晓 Google Play 政策[更新](https://support.google.com/googleplay/android-developer/answer/9934569?ref_topic=9877065&hl=zh-cn)，以确保您的 SDK 不会导致应用违反 Play 政策，例如[设备和网络滥用政策](https://support.google.com/googleplay/android-developer/answer/9888379?hl=zh-cn)、[广告政策](https://support.google.com/googleplay/android-developer/answer/9857753?ref_topic=9857752&hl=zh-cn)以及[用户数据政策中与永久标识符有关的规定](https://support.google.com/googleplay/android-developer/answer/10144311?hl=zh-cn)。否则，如有应用使用了您的 SDK，则可能会违反这些政策，并因此受到 Google Play 的违规处置。例如：
    
    - 如果您的 SDK 会使用个人数据和敏感的用户数据，您必须确保已在您公开提供的文档中向使用您 SDK 的应用开发者阐明这一点。
    - 如果 SDK 会在运行时加载 JavaScript、Python 或 Lua 等解释型语言（例如未打包到应用软件包中），则不得允许可能违反 Google Play 政策的行为，例如在没有正当理由，或未进行适当披露并征得用户同意的情况下收集已安装的软件包的数据。
    - 不出售个人信息以及敏感的用户信息。
- 在您的 SDK 中支持[最新的 API 安全功能和数据最少化功能](https://developer.android.com/google/play/requirements/target-sdk?hl=zh-cn)。如需了解详情，请参阅 [2022 年 4 月的博文](https://android-developers.googleblog.com/2022/04/expanding-plays-target-level-api-requirements-to-strengthen-user-security.html)。
    
- 帮助您的客户了解您的 SDK 可能会收集哪些用户数据以及它为何需要使用这些数据，以便应用开发者能够将相关说明添加到向最终用户提供的[醒目披露声明和用户意见征求](https://support.google.com/googleplay/android-developer/answer/10144311?hl=zh-cn)屏幕中，并能在适用的情况下将其添加到各自的隐私权政策中。
    
- 您应实现会读取并遵守应用开发者收集的用户偏好设置的逻辑，或者确保提供一种机制来让应用开发者根据这类面向用户的意见征求事件准确初始化您的 SDK。
    
- 以一种便于公开访问和浏览的格式提供您的数据使用方式信息。这里是一种[可选格式](https://support.google.com/googleplay/android-developer/answer/10787469?hl=zh-cn#optional_format_for_SDKs)，您或许会有兴趣使用它来发布信息，因为很多开发者都非常熟悉这种格式。如需查看示例，请参阅 [Google Firebase SDK 数据披露声明](https://support.google.com/analytics/answer/11582702?hl=zh-cn)和 [Google AdMob SDK 数据披露声明](https://developers.google.com/admob/android/play-data-disclosure?hl=zh-cn)。
