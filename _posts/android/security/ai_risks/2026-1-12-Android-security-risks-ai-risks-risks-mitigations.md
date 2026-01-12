---
title: Android-安全性-安全风险-AI-降低应用中的 AI 风险
date: 2026-1-12 14:57:58 +0800
categories:
  - Android
  - Security
  - Risks
  - OWASP
  - AI
tags:
  - Android
  - Security
  - Risks
  - OWASP
  - AI
description: 本指南为使用生成式 AI (GenAI) 构建 Android 应用的开发者提供了重要信息。生成式 AI 集成带来的独特挑战远超标准软件开发。我们会不断更新本指南，以反映新出现的 AI 风险以及不断变化的 AI 风险。 通过本内容，您可以了解将生成式 AI 功能集成到应用中的相关重大风险，并探索有效的缓解策略。
math: true
---
## 提示注入

[OWASP 风险说明](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)

攻击者精心设计恶意输入（提示），诱骗模型绕过其安全政策或执行意外操作。如果攻击成功，可能会导致模型泄露敏感数据、生成有害内容或执行意外操作，从而损害用户信任。了解如何防范[提示注入攻击](https://developer.android.com/privacy-and-security/risks/ai-risks/prompt-injection?hl=zh-cn)。

## 敏感信息披露

[OWASP 风险说明](https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/)

生成式 AI 模型会带来泄露敏感信息的重大风险，可能会泄露各种数据，从而影响 LLM 及其应用环境。当模型向其他用户输出此类机密数据时，当机器学习即服务 (MLaaS) 提供商不安全地存储提示时，或者当恶意行为者利用系统来揭示内部逻辑时，可能会发生泄露。后果非常严重，从知识产权侵犯和违规处罚（例如 [GDPR](https://gdpr-info.eu/)、[CCPA](https://oag.ca.gov/privacy/ccpa\))）到用户信任完全丧失和系统遭到入侵，不一而足。了解如何[降低敏感信息泄露](https://developer.android.com/privacy-and-security/risks/ai-risks/sensitive-data-disclosure?hl=zh-cn)风险。

## 过度的中介效应

[OWASP 风险说明](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/)

将 LLM 代理与函数或工具集成以执行操作可能会带来安全风险。利用函数调用的恶意提示可能会导致意外的数据丢失。开发者有责任实施安全措施（例如征得用户同意、验证、访问控制）来防止有害的 AI 操作。了解如何[降低代理机构风险过高的可能性](https://developer.android.com/privacy-and-security/risks/ai-risks/excessive-agency?hl=zh-cn)。

## 摘要

保障生成式 AI 系统的安全需要采取多方面的方法，将传统的信息安全实践扩展到 AI 模型带来的独特挑战，例如开发生命周期以及与用户和系统的互动。这项综合性工作包括主动测试、持续监控、完善的治理以及遵守不断变化的监管框架。将这些原则付诸实践的一个好方法是使用行业工具（例如 [SAIF 风险自评工具](https://saif.google/risk-self-assessment?hl=zh-cn)）来更好地了解和缓解风险。

