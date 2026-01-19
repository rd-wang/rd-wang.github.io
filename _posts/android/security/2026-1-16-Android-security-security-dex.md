---
title: Android-安全性-直接从 APK 运行嵌入式 DEX 代码
date: 2026-1-16 12:00:01 +0800
categories:
  - Android
  - Security
  - Security-Dex
tags:
  - Android
  - Security
  - Security-Dex
description: 在搭载 Android 10（API 级别 29）和更高版本的设备上，您现在可以告知平台直接从应用的 APK 文件运行嵌入式 DEX 代码。
math: true
---
在搭载 Android 10（API 级别 29）和更高版本的设备上，您现在可以告知平台直接从应用的 APK 文件运行嵌入式 DEX 代码。如果攻击者曾设法篡改了设备上本地编译的代码，此选项有助于防止这类攻击。

> **注意**：启用此功能可能会影响应用的性能，因为 ART 必须在应用启动时使用 [JIT 编译器](https://source.android.com/devices/tech/dalvik/jit-compiler?hl=zh-cn)（而不是读取提前编译好的原生代码）。我们建议您先测试应用性能，然后再决定是否在已发布的应用中启用此功能。

如果您使用的是 Gradle 构建系统，若要启用此功能，请执行以下操作：

- 在应用清单文件的 [`<application>`](https://developer.android.com/guide/topics/manifest/application-element?hl=zh-cn) 元素中将 `android::useEmbeddedDex` 属性设置为 `true`。
    
- 在模块级 `build.gradle.kts` 文件（如果您使用的是 Groovy，则为 `build.gradle` 文件）中将 `useLegacyPackaging` 设置为 `false`。
    
    > **注意**：如果您使用的 AGP 版本低于 4.2，请勿设置 `useLegacyPackaging` 选项。
    
    [Kotlin](https://developer.android.com/privacy-and-security/security-dex#kotlin-build.gradle.kts)
    
    ```
      packagingOptions {
        dex {
          useLegacyPackaging = false
        }
      }
    ```
      
    [Groovy](https://developer.android.com/privacy-and-security/security-dex#groovy-build.gradle)
    
    ```
      packagingOptions {
        dex {
          useLegacyPackaging false
        }
      }
    ```

如果您使用的是 Bazel 构建系统，若要启用此功能，请在应用清单文件的 `<application>` 元素中将 `android:useEmbeddedDex` 属性设置为 `true`，并让 DEX 文件保持未压缩状态：

android_binary(
   ...
   [nocompress_extensions](https://docs.bazel.build/versions/master/be/android.html?hl=zh-cn#android_binary.nocompress_extensions) = [".dex"],
)

