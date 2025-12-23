---
title: Android-安全风险-StrandHogg攻击/TaskAffinity漏洞
date: 2025-12-23 15:58:04 +0800
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
description: 恶意应用程序可以将其某个 Activity 的 taskAffinity 设置为与目标应用程序的packageName。随后与 intent 劫持相结合，会导致当用户下次启动目标应用时，恶意应用也同时启动并显示在目标应用之上。
math: true
---
**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

StrandHogg 攻击/任务相关性漏洞是因 Android 处理多个任务的方式（尤其是称为“更改父级任务”的功能）存在设计上的 bug 而产生/引起的。应用更改父级任务是一项功能，可让应用将某个 activity 从一个任务移至另一个任务。

StrandHogg 攻击利用了传入应用任务堆栈 activity 审查方式不明确的缺陷，致使恶意应用可以执行以下任一操作：

- 将恶意 activity 移入或移出受害堆栈
- 受害 activity 运行完毕后，将恶意 activity 设置为返回堆栈。

攻击者通过操纵 `allowTaskReparenting` 和 `taskAffinity` 设置来利用此漏洞。

## 影响

恶意应用程序可以将其某个 Activity 的 taskAffinity 设置为与目标应用程序的packageName。随后与 intent 劫持相结合，会导致当用户下次启动目标应用时，恶意应用也同时启动并显示在目标应用之上。

然后，攻击者就可以利用此Task Affinity漏洞劫持合法用户操作。

用户可能会被诱骗向恶意应用提供凭据。默认情况下，一旦 activity 启动并与 task 关联，该关联关系将持续到活动的整个生命周期结束。但是，将 allowTaskReparenting 设置为 true 可以打破此限制，允许将现有活动重新设置为新创建的“原生”任务的父级。

例如，App A 可以被 App B 定位，App A 的 activity 在完成返回后重定向到 App B 的 activity 堆栈。
这种从一个应用程序到另一个应用程序的切换对用户来说是隐藏的，这会造成严重的网络钓鱼威胁。
## 缓解措施

更新为 `android:minSdkVersion="30"`.

StrandHogg 攻击/任务亲和性漏洞最初于 2019 年 3 月进行了修复，更新、更全面的变种于 2020 年 9 月进行了修复。Android SDK 版本 30 及更高版本（Android 11）包含相应的操作系统补丁，可以避免此漏洞。虽然可以通过个别应用程序配置来部分缓解 StrandHogg 攻击的第 1 版，但只有通过此 SDK 版本补丁才能阻止攻击的第 2 版。

## 资源

- [2015 年 Usenix 大会中探讨此漏洞的原创学术论文](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-ren-chuangang.pdf)
- [Promon 安全团队对此原始漏洞的详细说明](https://promon.co/security-news/the-strandhogg-vulnerability/)

