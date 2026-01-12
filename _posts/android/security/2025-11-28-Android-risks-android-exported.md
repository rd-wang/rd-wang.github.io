---
title: Android-安全性-安全风险-android:exported
date: 2025-11-28 11:22:04 +0800
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
description: 设置某个组件（activity、服务、广播接收器等）是否可以由其他应用的组件启动
math: true
---


# android:exported

**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

`android:exported` [属性](https://developer.android.com/guide/topics/manifest/activity-element?hl=zh-cn#exported)用于设置某个组件（activity、服务、广播接收器等）是否可以由其他应用的组件启动：

- 如果设为 `true`，则任何应用都可以访问相应的 activity，并通过其确切类名称启动它。
- 如果设为 `false`，则只有同一应用的组件、具有相同用户 ID 的应用或具有特权的系统组件可以启动该 activity。

此属性的默认值背后的逻辑会随时间的推移而发生变化，且会因组件类型和 Android 版本而异。例如，在 API 级别 16 (Android 4.1.1) 或更低版本中，`<provider>` 元素的值默认设为 `true`。如果未明确设置此属性，便存在某些设备之间具有不同默认值的风险。

## 影响

具有不同默认值的情况意味着可能会意外地公开内部应用组件。下面列出了几种后果作为示例：

拒绝服务攻击。 其他应用通过不当方式访问内部组件以修改应用的内部功能。 敏感数据泄露。 在易受攻击的应用环境中执行代码。

## 缓解措施

始终明确设置 `android:exported` 属性。这样便不必进行任何解释，因为已经明确表达了您对组件可见性的意图。
