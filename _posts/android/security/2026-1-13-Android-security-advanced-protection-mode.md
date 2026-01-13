---
title: Android-安全性-高级保护模式
date: 2026-1-13 8:25:16 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Protection-Mode
description: Android 高级保护模式 (AAPM) 是一项新功能，旨在为面临风险的用户增强 Android 设备的安全性
math: true
---
Android 高级保护模式 (AAPM) 是一项新功能，旨在为面临风险的用户增强 Android 设备的安全性。它是一个单一设置，实施一组预先确定的配置，旨在加强设备保护。AAPM 将安全性置于某些可能降低的功能和可用性之上，这意味着某些功能可能会受到限制，以最大限度地减少攻击面。

## 影响

以下内容介绍了对开发者的影响：

- 功能：AAPM 作为一个单一设置运行，可激活一系列安全配置，旨在增强对处于风险中的用户设备的保护。这将对某些服务的行为带来改变，应用程序开发人员需要对此进行调整。
- 向已订阅应用发送信号：当用户启用 AAPM 时，系统会向所有已订阅的应用发送信号。此信号通知这些应用程序，使其适应 AAPM 启用的功能的改变行为。
- 应用修改：订阅应用的开发者必须修改其应用，以符合 AAPM 触发的行为变更。 此类修改的示例包括：
    - 调整应用程序逻辑以适应禁用 2G 和 WEP 网络连接。
    - 修改应用行为以符合防止侧载的要求。
    - 适应取证日志记录功能。
    - 由于屏蔽了未知号码的来电，因此调整了与通话处理相关的功能。
    - 集成或兼容即时通讯应用内链接的垃圾邮件防护机制。
    - 包括应用程序开发商采取的额外缓解措施，以进一步保护高风险用户。
- 目标受众：AAPM 预计主要影响那些包含针对具有较高安全意识的用户量身定制的安全功能的应用程序。当用户选择 AAPM 时，这些应用程序将受益于自动激活。

## 与 AAPM 集成

如需使用相关 API，需要声明以下权限

```xml
<uses-permission android:name="android.permission.QUERY_ADVANCED_PROTECTION_MODE" />
```

以下 API 来自新推出的 `AdvancedProtectionManager` 系统服务。

```java
public class AdvancedProtectionManager() {
  // Check the current status
  public boolean isAdvancedProtectionEnabled();

  // Be alerted when status changes
  public void registerAdvancedProtectionCallback(Executor executor, Callback callback);

  public void unregisterAdvancedProtectionCallback(Callback callback);
}

public class Callback() {
  // Called when advanced protection state changes
  void onAdvancedProtectionChanged(boolean enabled);
}
```

>**注意**： 当应用终止时，其注册的回调会被移除。 由于已终止的应用无法恢复并接收 AAPM 状态更改，因此最好在应用的初始化阶段注册回调。此外，在初始化期间执行按需 AAPM 状态查询，以确保您拥有当前状态。