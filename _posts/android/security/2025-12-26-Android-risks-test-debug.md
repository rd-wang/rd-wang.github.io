---
title: Android-安全风险-测试和调试功能
date: 2025-12-26 08:30:07 +0800
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
description: 恶意用户在生产版本中与测试或调试功能交互可能会导致意想不到的后果。任何操作的影响都与分配给该功能的权限直接相关。权限越高，主动利用漏洞造成的影响就越大。应用程序内的此类功能可用于绕过多种保护措施、绕过付费墙、检索系统或用户相关信息，或触发测试活动。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

发布包含测试或调试功能的生产版本可能会对应用程序的安全状况产生负面影响。这些功能用于帮助开发人员在新版本发布之前或之后，在预期的应用程序使用场景中发现和识别错误，不应公开访问。

测试/调试功能的示例如下：

- 隐藏的菜单
- 用于启用调试日志的选项
- 用于更改应用流程的选项
- 用于绕过付款或订阅流程的选项
- 用于绕过身份验证的选项
- 针对应用程序特定活动的测试

恶意用户可以利用上述所有手段来改变应用程序的预期流程或检索系统信息，从而定制进一步的攻击。

暴露的测试或调试功能所带来的风险可能因与调试功能本身相关的操作而异。

应用面临的另一个风险区域是在 AndroidManifest.xml 元素 **`<application>`** 中设置的 **android:debuggable** 属性。如果部署的正式版应用设置了 [**android:debuggable**](https://developer.android.com/topic/security/risks/android-debuggable?hl=zh-cn) 一文中所述的值，那么恶意用户便可访问原本无法访问的管理资源。

## 影响

恶意用户在生产版本中与测试或调试功能交互可能会导致意想不到的后果。任何操作的影响都与分配给该功能的权限直接相关。权限越高，主动利用漏洞造成的影响就越大。应用程序内的此类功能可用于绕过多种保护措施、绕过付费墙、检索系统或用户相关信息，或触发测试活动。

## 缓解措施

### 避免使用调试组件

测试或调试功能绝不应在生产应用程序组件（例如 activities, broadcast receivers, services or content providers）中实现，因为如果导出，则设备上的任何其他进程都可以运行这些功能。将调试组件设置为未导出 ([**android:exported="false"**](https://developer.android.com/topic/security/risks/android-exported?hl=zh-cn)）并不构成对这些功能的有效保护，因为如果启用了调试选项，已取得 root 权限的设备仍然可以通过 Android Debug Bridge (ADB) 工具来执行调试功能。

### 限制调试或测试功能仅在预发布版本中使用

应用程序内任何测试或调试功能的执行都应仅限于一组受限的暂存版本，以便仅允许开发人员在受控环境中调试或测试应用程序的功能。这可以通过创建应用程序的专用测试或调试版本，并为其编写高级检测测试来实现，以确保任何测试或调试功能都在隔离的版本上运行。

### 实现自动化界面测试

在对应用程序进行测试时，选择自动化 UI 测试，因为它们可重复执行，可以在独立的环境中执行，并且不易出现人为错误。

## 资源

- [关于高级测试设置的开发者指南](https://developer.android.com/studio/test/advanced-test-setup?hl=zh-cn)
- [关于自动执行界面测试的开发者指南](https://developer.android.com/training/testing/instrumented-tests/ui-tests?hl=zh-cn)
- [android:debuggable](https://developer.android.com/topic/security/risks/android-debuggable?hl=zh-cn)
- [android:exported](https://developer.android.com/topic/security/risks/android-exported?hl=zh-cn)
- [Android 市场中的可调试应用](https://labs.withsecure.com/publications/debuggable-apps-in-android-market)
- [调试代码会导致安全漏洞吗？](https://www.coderskitchen.com/an-debug-code-cause-security-vulnerabilities/)

