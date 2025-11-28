---
title: Android-安全风险-android:debuggable
date: 2025-11-28 11:06:24 +0800
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
description:
math: true
---


# android:debuggable

**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

`android:debuggable` [属性](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn)用于设置应用是否可调试。它为整个应用进行设置，不能被个别组件替换。此属性默认设为 `false`。

如果允许应用可调试，这本身不是漏洞，但这样做意味着允许用户在未经授权的情况下使用管理功能，从而导致应用面临更大的风险。这样一来，攻击者可能会比预期更容易访问该应用及其所用的资源。

## 影响

如果您将 `android:debuggable` 标志设置为 true，攻击者就能够调试该应用，而这会使他们更容易访问应用中的“安全重地”。

## 缓解措施

交付应用时，请务必将 `android:debuggable` 标志设置为 `false`。