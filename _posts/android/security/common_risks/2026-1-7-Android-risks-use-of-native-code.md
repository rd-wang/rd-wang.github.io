---
title: Android-安全性-安全风险-使用原生代码
date: 2026-1-7 09:32:46 +0800
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
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

Android 应用可以利用使用 C 和 C++ 等语言编写的原生代码来实现特定功能。不过，当应用使用 Java 原生接口 (JNI) 与此原生代码交互时，可能会暴露自身的漏洞，例如缓冲区溢出和原生代码实现中可能存在的其他问题。

## 影响

尽管在 Android 应用中使用原生代码有许多积极影响（例如性能优化和混淆），但也可能会对安全性产生负面影响。C/C++ 等原生代码语言缺少 Java/Kotlin 的内存安全功能，因此容易受到缓冲区溢出、释放后使用错误和其他内存损坏问题等漏洞的影响，从而导致崩溃或任意代码执行。此外，如果原生代码组件存在漏洞，则可能会危及整个应用，即使其余部分均以 Java 安全编写也是如此。

## 缓解措施

### 开发和编码指南

- **安全编码准则**：对于 C/C++ 项目，请遵循已建立的安全编码标准（例如 CERT、OWASP 等）来减少缓冲区溢出、整数溢出和格式字符串攻击等漏洞。优先考虑以质量和安全性而闻名的 Abseil 等库。请尽可能考虑采用 Rust 等内存安全型语言，它们的性能与 C/C++ 相当。
- **输入验证**：严格验证从外部来源收到的所有输入数据，包括用户输入、网络数据和文件，以防止注入攻击和其他漏洞。

### 强化编译选项

通过激活堆栈保护 (Canary)、重定位只读 (RELRO)、数据执行防护 (NX) 和位置无关可执行文件 (PIE) 等保护机制，可以利用 ELF 格式的原生库针对各种漏洞进行安全强化。为方便起见，Android NDK 编译选项已默认启用所有这些保护。

如需验证这些安全机制在二进制文件中的实现情况，您可以使用 `hardening-check` 或 `pwntools` 等工具。

[Bash](https://developer.android.com/privacy-and-security/risks/use-of-native-code?hl=zh-cn#bash)

```bash
$ pwn checksec --file path/to/libnativecode.so
    Arch:     aarch64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

### 验证第三方库是否存在漏洞

选择第三方库时，应优先使用在开发者社区中拥有良好声誉的库。[Google Play SDK 索引](https://play.google.com/sdks?hl=zh-cn)等资源可帮助您找到广受好评且值得信赖的库。确保将库更新到最新版本，并使用 [Exploit-DB](https://www.exploit-db.com/) 中的数据库等资源主动搜索与这些库相关的任何已知漏洞。使用 `[library_name] vulnerability` 或 `[library_name] CVE` 等关键字进行网络搜索可能会泄露重要的安全信息。

## 资源

- [CWE-111：直接使用不安全的 JNI](https://cwe.mitre.org/data/definitions/111.html)
- [利用数据库](https://www.exploit-db.com/)
- [检查二进制文件是否具有安全增强功能](https://www.systutorials.com/docs/linux/man/1-hardening-check/)
- [使用 pwntools 检查二进制安全设置](https://docs.pwntools.com/en/stable/commandline.html#pwn-checksec)
- [Linux 二进制文件安全强化](https://medium.com/@n80fr1n60/linux-binary-security-hardening-1434e89a2525)
- [使用重定位只读 (RELRO) 强化 ELF 二进制文件](https://www.redhat.com/fr/blog/hardening-elf-binaries-using-relocation-read-only-relro)
- [OWASP 二进制保护机制](https://mas.owasp.org/MASTG/Android/0x05i-Testing-Code-Quality-and-Build-Settings/#binary-protection-mechanisms)
- [SEI CERT 编码标准](https://wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards)
- [OWASP 开发者指南](https://owasp.org/www-project-developer-guide/release/)
- [Google Play SDK 索引](https://play.google.com/sdks?hl=zh-cn)
- [Android NDK](https://developer.android.com/ndk?hl=zh-cn)
- [Android Rust 简介](https://source.android.com/docs/setup/build/rust/building-rust-modules/overview?hl=zh-cn)
- [Abseil（C++ 通用库）](https://github.com/abseil/abseil-cpp)
- [PIE 由链接器强制执行](https://cs.android.com/android/platform/superproject/main/+/main:bionic/linker/linker_main.cpp;l=425?q=linker_main&%3Bss=android%2Fplatform%2Fsuperproject%2Fmain&hl=zh-cn)

