---
title: Android-安全性-安全风险-点按劫持
date: 2025-12-25 16:24:21 +0800
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
description: 恶意应用通过添加叠加层来遮盖界面的方式或通过其他方式，诱骗用户点击与安全相关的控件（确认按钮等）
math: true
---
**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

“点按劫持”发生在 Android 应用中，是与[点击劫持 Web 漏洞](https://owasp.org/www-community/attacks/Clickjacking)类似的攻击：恶意应用通过添加叠加层来遮盖界面的方式或通过其他方式，诱骗用户点击与安全相关的控件（确认按钮等）。在本页中，我们将对以下两种攻击变体加以区分：完全遮盖和部分遮盖。在完全遮盖攻击中，攻击者会为触摸区域添加叠加层；而在部分遮盖攻击中，触摸区域不会被遮盖。

## 影响

点按劫持攻击用于诱骗用户执行特定操作。具体影响取决于攻击者想要诱骗用户执行的操作。

## 风险：完全遮盖

在完全遮盖攻击中，攻击者会为触摸区域添加叠加层，以便劫持触摸事件：
![full_occlusion](https://rd-wang.github.io/assets/img/android/risk/full_occlusion.png)

### 缓解措施

在代码中设置 `View.setFilterTouchesWhenObscured(true)` 可防止完全遮盖。这会屏蔽叠加层传递的触摸事件。如果您更喜欢采用声明式方法，还可以在布局文件中为要保护的 [`View`](https://developer.android.com/reference/android/view/View?hl=zh-cn#security) 对象添加 `android:filterTouchesWhenObscured="true"`。

>**注意**：默认情况下，Android S（12，SDK 31）及更高版本会屏蔽来自其他 UID 的不受信任的叠加层的触摸事件，以防止完全遮盖攻击。  
  不过，有一点需要注意：对于系统提醒窗口 (SAW) 和窗口动画，只有不透明度 >= 0.8 的图层的触摸才会被阻止。这样做的理由是 SAW 需要用户授予权限，而阻止所有限时动画的事件可能会损害用户体验。

---

## 风险：部分遮盖

在部分遮盖攻击中，触摸区域不会受到遮盖：

![partial_occlusion](https://rd-wang.github.io/assets/img/android/risk/partial_occlusion.png)

### 缓解措施

通过手动忽略带有 `FLAG_WINDOW_IS_PARTIALLY_OBSCURED` 标志的触摸事件来缓解部分遮挡问题。默认情况下，系统没有针对这种情况的保护措施。

>**注意：潜在风险**：此缓解措施可能会干扰一些良性应用。在某些情况下，由于部分遮挡是由良性应用引起的，因此可能无法部署此修复程序，因为这会对用户体验产生负面影响。

Android 16 和 `accessibilityDataSensitive`：从 Android 16（API 级别 16）及更高版本开始，开发者可以使用 `accessibilityDataSensitive`标志来进一步保护敏感数据免受恶意辅助功能服务的侵害，这些服务并非合法的辅助功能工具。当在敏感视图（例如登录屏幕、交易确认屏幕）上设置此标志时，除非应用在其清单文件中声明 `isA11yTool=true`，否则拥有辅助功能权限的应用将无法读取或与敏感数据交互。这提供了一种更强大的系统级保护，可以抵御部分遮挡场景下常见的窃听和点击注入攻击。开发者通常可以通过在布局文件中指定 `android:filterTouchesWhenObscured="true"` 来隐式启用 `accessibilityDataSensitive`。


---

## 具体风险

本节收集了需要非标准缓解策略或在某些 SDK 级别中已缓解的风险，此处仅供参考。

### 风险：android.Manifest.permission.SYSTEM_ALERT_WINDOW

[`SYSTEM_ALERT_WINDOW`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SYSTEM_ALERT_WINDOW) 权限允许应用创建一个显示在所有应用之上的窗口。
#### 缓解措施

较新版本的 Android 引入了几种缓解措施，其中包括：

- 在 Android 6（API 级别 23）及更高版本中，用户必须明确向应用授予创建叠加窗口的权限。
- 在 Android 12（API 级别 31）及更高版本中，应用可以将 `true` 传递到 [`Window.setHideOverlayWindows()`](https://developer.android.com/reference/android/view/Window?hl=zh-cn#setHideOverlayWindows\(boolean\))。

---

### 风险：自定义消息框

攻击者可以使用 `Toast.setView()` 自定义[消息框](https://developer.android.com/guide/topics/ui/notifiers/toasts?hl=zh-cn)的外观。在 Android 10（API 级别 29）及更低版本中，恶意应用可能会从后台启动此类消息框。

#### 缓解措施

如果应用以 Android 11（API 级别 30）或更高版本为目标平台，系统会屏蔽后台自定义消息框。不过，在某些情况下，可以通过“消息框爆发”的手段规避这种缓解措施：攻击者趁应用位于前台时将多个消息框列入队列，这样一来，即使应用进入后台，这些消息框也会继续启动。

从 Android 12（API 级别 31）开始，后台消息框攻击和消息框爆发攻击已彻底得到缓解。

---

### 风险：activity sandwich

如果恶意应用成功诱使用户打开它，它仍然可以启动受害应用中的 activity，然后将其自己的activity覆盖在上面，从而形成“activity sandwich”，并实现部分遮盖攻击。

#### 缓解措施

查看适用于部分遮盖攻击的常规缓解措施。为加强防御，请确保您不会导出原本不需要导出的 activity，以防止攻击者对其做 sandwich 处理。

---

## 资源

- [“Play 商店目标 API 级别”政策变更](https://android-developers.googleblog.com/2019/02/expanding-target-api-level-requirements.html)
- [界面伪装和基于无障碍服务的 Android 攻击](https://cloak-and-dagger.org/)

