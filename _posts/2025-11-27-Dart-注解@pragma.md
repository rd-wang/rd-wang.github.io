---
title: Dart-注解@pragma
date: 2025-11-27 9:04:57 +0800
categories:
  - Dart
tags:
  - Dart
  - Flutter
description: "Dart的`@pragma(vm:entry-point, ...)`注解，用于指示AOT预编译器处理从本机代码直接访问的类、方法和字段。该注解对于确保预编译时的正确性和混淆后的符号保留至关重要。内容涵盖了语法、应用场景，以及如何在类、方法和字段上使用该注解。"
math: true
---
> [Dart语言详解](https://rd-wang.github.io/posts/Dart-基础概念和内部原理/)

# `vm:entry-point` 含义
注解`@pragma("vm:entry-point", ...)` 必须放在类或成员上，以表明它可以在 AOT 模式下直接从本机或 VM 代码解析、分配或调用。

### 背景
Dart VM 预编译器（AOT 编译器）执行整个程序优化，例如摇树和类型流分析 (TFA)，以减少生成的编译应用程序的大小并提高其性能。这种优化假设编译器可以看到整个 Dart 程序，并且能够发现和分析所有可能在运行时执行的 Dart 函数和成员。虽然 Dart 代码完全可用于预编译器，但嵌入器的本机代码和本机方法是编译器无法访问的。这样的原生代码可以通过原生 Dart API 回调到 Dart。

为了引导预编译器，程序员必须明确列出入口点（roots）——从本机代码访问的 Dart 类和成员。请注意，列出入口点不是可选的：只要程序定义了调用 Dart 的本地方法，入口点是编译正确性所必需的。

此外，当启用混淆时，预编译器需要知道需要保留哪些符号以确保可以从本机代码中解析它们。

### Syntax
注释的允许用途如下。

### Classes
以下任何一种形式都可以附加到一个类中：
```
@pragma("vm:entry-point")
@pragma("vm:entry-point", true/false)
@pragma("vm:entry-point", !const bool.formEnvironment("dart.vm.product"))
class C { ... }
```
如果缺少第二个参数，null或者true，该类将可直接从本机或 VM 代码分配。

`@pragma("vm:entry-point")`可能会添加到抽象类中——在这种情况下，它们的名称会在混淆后幸存下来，但它们不会有任何分配存根。

### Procedures
以下任何一种形式都可以附加到Procedures（包括 getter、setter 和构造函数）：
```
@pragma("vm:entry-point")
@pragma("vm:entry-point", true/false)
@pragma("vm:entry-point", !const bool.formEnvironment("dart.vm.product"))
@pragma("vm:entry-point", "get")
@pragma("vm:entry-point", "call")
void foo() { ... }
```
如果缺少第二个参数，null或者true，则Procedures（及其封闭形式，不包括构造函数和设置器）将可用于直接从本机或 VM 代码查找和调用。

如果Procedures是生成构造函数，则还必须对封闭类进行注释，以便从本机或 VM 代码进行分配。

如果注释是“get”或“call”，则该Procedures将仅可用于闭包（通过 访问`Dart_GetField`）或调用（通过访问 `Dart_Invoke`）。

"@pragma("vm:entry-point", "get") 不允许针对构造函数或设置器，因为它们不能被封闭。

### Fields
以下任何一种形式都可以附加到非静态字段。前三个表单可以附加到静态字段。

```
@pragma("vm:entry-point")
@pragma("vm:entry-point", null)
@pragma("vm:entry-point", true/false)
@pragma("vm:entry-point", !const bool.formEnvironment("dart.vm.product"))
@pragma("vm:entry-point", "get"/"set")
int foo;
```
如果缺少第二个参数null或“true”，则该字段被标记为本地访问，对于非静态字段，封闭类的接口中相应的 getter 和 setter 被标记为本地调用。如果使用 'get'/'set' 参数，则只标记 getter/setter。对于静态字段，始终标记隐式 getter。第三种形式对静态字段没有意义，因为它们不属于接口。

任何形式的入口点注释都不允许调用字段。
