---
title: Android-安全性-reCAPTCHA验证恶意流量
date: 2026-1-15 11:25:24 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Safety-Net
description: 使用高级风险分析引擎来保护您的应用免受网络垃圾和其他滥用行为的影响。如果该服务怀疑与您的应用互动的用户可能是机器人而非真人，则会提供一项人机识别系统验证，真人必须先通过这一验证，然后您的应用才可以继续执行。
math: true
---
>为了帮助安全地发展业务，Google 开发了各种工具来保护您的 Android 应用和游戏免受滥用。我们会随着滥用形势的变化不断改进这些解决方案。
>为了兑现这一承诺，谷歌正在用 [reCAPTCHA]( https://cloud.google.com/recaptcha/docs/instrument-android-apps取代) SafetyNet reCAPTCHA API。reCAPTCHA 为移动应用程序提供更强大的安全保护。
>我们计划从 2025 年第三季度开始逐步停止 SafetyNet reCAPTCHA API 的使用。详情请参阅  [reCAPTCHA 弃用和关闭政策](https://cloud.google.com/recaptcha/docs/deprecation-policy-mobile)。

SafetyNet 服务包含一种 reCAPTCHA API，该 API 可用于防止您的应用遭受恶意流量的侵害。

reCAPTCHA 是一项免费服务，它会使用高级风险分析引擎来保护您的应用免受网络垃圾和其他滥用行为的影响。如果该服务怀疑与您的应用互动的用户可能是机器人而非真人，则会提供一项人机识别系统验证，真人必须先通过这一验证，然后您的应用才可以继续执行。

> **注意：** 如需使用此 API，您的应用必须将 `AndroidManifest.xml` 文件中的 `minSdkVersion` 设置为 `14` 或更高版本。

本文档介绍了如何将 SafetyNet 中的 reCAPTCHA API 集成到您的应用中。

## 附加服务条款

访问或使用 reCAPTCHA API 即表示您同意接受 [Google API 服务条款](https://developers.google.com/terms/?hl=zh-cn)和以下 reCAPTCHA 服务条款。请先阅读并了解所有适用的条款及政策，然后再使用这些 API。

### reCAPTCHA 服务条款

您承认并了解，reCAPTCHA API 在运行时会收集各种软硬件信息，例如设备和应用数据以及完整性检查的结果等，并会将这些数据发送给 Google 进行分析。根据 Google API 服务条款第 3(d) 条的规定，您同意使用这些 API 即表示您负责就收集此类数据以及与 Google 共享此类数据提供任何必要的通知或同意。

## 注册 reCAPTCHA 密钥对

如需注册用于 SafetyNet reCAPTCHA API 的密钥对，请转到 [reCAPTCHA Android 注册网站](https://www.google.com/recaptcha/admin/create?hl=zh-cn)，然后完成下面这一系列步骤：

1. 在显示的表单中提供以下信息：
    
    - **标签：** 您的密钥的唯一标签。通常情况下，您可以使用公司或单位的名称。
    - **reCAPTCHA 类型：** 选择  **reCAPTCHA v2**，然后选择 **reCAPTCHA Android**。
    - **软件包：** 请提供使用此 API 密钥的每个应用的软件包名称。为了让应用使用该 API，您输入的软件包名称必须与该应用的软件包名称完全一致。输入时，请将每个软件包名称单独放在一行。
    - **所有者：** 为贵组织中负责监控应用 reCAPTCHA 评估的每个人添加电子邮件地址。
2. 选中**接受“reCAPTCHA 服务条款”**复选框。
    
3. **向所有者发出提醒：** 如果您想接收有关此 reCAPTCHA API 的电子邮件，请选中此复选框，然后点击**提交**。
    
4. 在随即显示的页面中，您可以在**网站密钥**和**密钥**下分别找到您的公钥和私钥。您可以在[发送验证请求](https://developer.android.com/privacy-and-security/safetynet/recaptcha?hl=zh-cn#send-request)时使用网站密钥，在[验证用户响应令牌](https://developer.android.com/privacy-and-security/safetynet/recaptcha?hl=zh-cn#validate-response)时使用密钥。
    

## 添加 SafetyNet API 依赖项

请先将 SafetyNet API 添加到您的项目，然后才能使用 reCAPTCHA API。如果您使用的是 Android Studio，请将此依赖项添加到您的应用级 Gradle 文件。如需了解详情，请参阅 [SafetyNet API 设置](https://developer.android.com/training/safetynet?hl=zh-cn#before-you-begin)。

## 使用 reCAPTCHA API

本部分介绍了如何调用 reCAPTCHA API 以发送人机识别系统验证请求并接收用户响应令牌。

### 发送验证请求

如需调用 SafetyNet reCAPTCHA API，您需要调用 [`verifyWithRecaptcha()`](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetClient.html?hl=zh-cn#verifyWithRecaptcha\(java.lang.String\)) 方法。通常情况下，此方法与用户在您的 Activity 中选择界面元素（如按钮）相对应。

在您的应用中使用 `verifyWithRecaptcha()` 方法时，您必须执行以下操作：

- 将您的 API 网站密钥作为参数传入。
- 替换 [`onSuccess()`](https://developers.google.com/android/reference/com/google/android/gms/tasks/OnSuccessListener.html?hl=zh-cn#onSuccess\(TResult\)) 和 [`onFailure()`](https://developers.google.com/android/reference/com/google/android/gms/tasks/OnFailureListener.html?hl=zh-cn#onFailure\(java.lang.Exception\)) 方法，以便处理验证请求任务的两种可能结果。具体来说，如果 API 将 [`ApiException`](https://developers.google.com/android/reference/com/google/android/gms/common/api/ApiException?hl=zh-cn) 的实例传入 `onFailure()`，您需要处理使用 [`getStatusCode()`](https://developers.google.com/android/reference/com/google/android/gms/common/api/ApiException?hl=zh-cn#getStatusCode\(\)) 可检索到的每个可能的状态代码。

以下代码段演示了如何调用此方法：

[Kotlin](https://developer.android.com/privacy-and-security/safetynet/recaptcha?hl=zh-cn#kotlin)
```kotlin
fun onClick(view: View) {
    SafetyNet.getClient(this).verifyWithRecaptcha(YOUR_API_SITE_KEY)
            .addOnSuccessListener(this as Executor, OnSuccessListener { response ->
                // Indicates communication with reCAPTCHA service was
                // successful.
                val userResponseToken = response.tokenResult
                if (response.tokenResult?.isNotEmpty() == true) {
                    // Validate the user response token using the
                    // [reCAPTCHA siteverify API](https://developers.google.com/recaptcha/docs/verify).
                }
            })
            .addOnFailureListener(this as Executor, OnFailureListener { e ->
                if (e is ApiException) {
                    // An error occurred when communicating with the
                    // reCAPTCHA service. [Refer to the status code](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetStatusCodes) to
                    // handle the error appropriately.
                    Log.d(TAG, "Error: ${CommonStatusCodes.getStatusCodeString(e.statusCode)}")
                } else {
                    // A different, unknown type of error occurred.
                    Log.d(TAG, "Error: ${e.message}")
                }
            })
}
```
[Java](https://developer.android.com/privacy-and-security/safetynet/recaptcha?hl=zh-cn#java)
```java

public void onClick(View v) {
    SafetyNet.getClient(this).verifyWithRecaptcha(YOUR_API_SITE_KEY)
        .addOnSuccessListener((Executor) this,
            new OnSuccessListener<SafetyNetApi.RecaptchaTokenResponse>() {
                @Override
                public void onSuccess(SafetyNetApi.RecaptchaTokenResponse response) {
                    // Indicates communication with reCAPTCHA service was
                    // successful.
                    String userResponseToken = response.getTokenResult();
                    if (!userResponseToken.isEmpty()) {
                        // Validate the user response token using the
                        // [reCAPTCHA siteverify API](https://developers.google.com/recaptcha/docs/verify?hl=zh-cn).
                    }
                }
        })
        .addOnFailureListener((Executor) this, new OnFailureListener() {
                @Override
                public void onFailure(@NonNull Exception e) {
                    if (e instanceof ApiException) {
                        // An error occurred when communicating with the
                        // reCAPTCHA service. [Refer to the status code](https://developers.google.com/android/reference/com/google/android/gms/safetynet/SafetyNetStatusCodes?hl=zh-cn) to
                        // handle the error appropriately.
                        ApiException apiException = (ApiException) e;
                        int statusCode = apiException.getStatusCode();
                        Log.d(TAG, "Error: " + CommonStatusCodes
                                .getStatusCodeString(statusCode));
                    } else {
                        // A different, unknown type of error occurred.
                        Log.d(TAG, "Error: " + e.getMessage());
                    }
                }
        });
}
```

### 验证用户响应令牌

如果 reCAPTCHA API 执行 `onSuccess()` 方法，则表示用户已成功完成人机识别系统验证。不过，此方法仅表示用户已正确通过人机识别系统验证。您仍需要从后端服务器验证用户的响应令牌。

如需了解如何验证用户的响应令牌，请参阅[验证用户的响应](https://developers.google.com/recaptcha/docs/verify?hl=zh-cn)。

