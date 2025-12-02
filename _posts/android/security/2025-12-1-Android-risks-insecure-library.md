---
title: Android-安全风险-不安全的API或库
date: 2025-12-1 08:35:35 +0800
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
description: 利用存在漏洞的依赖项可能造成数据泄露或服务不可用等后果
math: true
---

# 不安全的 API 或库

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

使用不安全的 API 或库会显著降低应用的安全状况。任何这些依赖项中的安全漏洞都可能使攻击者利用多种途径进行广泛的攻击，例如中间人攻击 (MitM) 和远程代码执行 (RCE)。

如果开发者未将安全评估和漏洞测试集成到软件开发生命周期 (SDLC)，或者在某些情况下未对应用依赖项实施自动更新政策，就会出现实施不安全依赖项的风险；

利用依赖项攻击，通常从分析应用程序二进制文件（.apk）开始，以查找存在漏洞的库。此时，系统会执行开源情报 (OSINT)，以找出之前发现的可利用漏洞。之后，攻击者便可利用公开披露的漏洞信息（例如常见漏洞和风险 [CVE]）执行进一步攻击。

## 影响

成功利用不安全的依赖项可能会导致各种攻击，例如远程代码执行 (RCE)、SQL 注入 (SQLi) 或跨站脚本攻击 (XSS)。因此，总体影响与第三方软件引入的漏洞类型以及攻击者可利用的漏洞类型直接相关。成功利用存在漏洞的依赖项可能造成数据泄露或服务不可用等后果，这可能会对声誉和经济收入产生重大影响。

## 缓解措施

### 纵深防御

请注意，下列缓解措施必须结合起来实施，以确保强化安全状况，并缩小应用的攻击面。始终应根据具体情况评估确切的方法。

### 依赖项漏洞评估

在开发生命周期之初实施依赖项验证，以检测第三方代码中的漏洞。此阶段会在将非内部构建的代码发布到生产环境之前先测试这些代码是否安全。除了验证之外，还可以在软件开发生命周期内辅以实现静态应用安全保障测试 (SAST) 和动态应用安全保障测试 (DAST) 工具，以改善应用的安全状况。

### 持续更新依赖项

始终要注意持续更新代码中嵌入的任何依赖项。为此，我们建议，只要第三方组件发布新的安全补丁，便实现推送到生产环境的自动更新。

### 执行应用渗透测试

执行常规渗透测试。此类测试旨在发现任何可能影响专有代码和/或第三方依赖项的已知漏洞。此外，安全评估还会经常发现未知漏洞 (0-day)。渗透测试对开发者来说很实用，因为这些测试能为他们提供应用当前安全状况的概况信息，并帮助他们确定必须解决的可能被利用的安全问题的优先级。

## 资源

- [如何识别和管理不安全的依赖项](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html)
    
- [GitHub 安全功能](https://docs.github.com/en/code-security/getting-started/github-security-features)
    
- [如何保护依赖项](https://www.hacksplaining.com/prevention/toxic-dependencies)
    
- [CWE-1395：存在漏洞的第三方组件的依赖项](https://cwe.mitre.org/data/definitions/1395.html)
    
- [适用于 Android 的 SDK 实现最佳实践](https://developer.android.com/guide/practices/sdk-best-practices?hl=zh-cn)。
