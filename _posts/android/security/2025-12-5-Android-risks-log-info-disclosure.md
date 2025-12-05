---
title: Android-安全风险-日志信息泄露
date: 2025-12-5 14:19:33 +0800
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
description: 日志信息泄露这类漏洞的严重程度可能会因敏感数据的上下文和类型而异。总体而言，这类漏洞造成的影响就是会使个人身份信息和凭据等潜在关键信息失去机密性。
math: true
---
# 日志信息泄露

**OWASP 类别：**[MASVS-STORAGE：存储](https://mas.owasp.org/MASVS/05-MASVS-STORAGE)

## 概览

_日志信息泄露_是一种漏洞，应用程序会将敏感数据写入设备日志。如果这些信息落入恶意攻击者手中，可能极具价值，例如用户的凭据或个人身份信息 (PII)，或者可能被用于发起进一步的攻击。

以下任何一种情况都可能出现此问题：

- 应用程序生成的日志：
    - 日志有意允许未经授权的人员访问，但却意外地包含了敏感数据。
    - 日志中故意包含敏感数据，但却意外地被未经授权的人员访问。
    - 通用错误日志，有时可能会打印敏感数据，具体取决于触发的错误消息。
- 外部生成的日志：
    - 外部组件负责打印包含敏感数据的日志。

Android `Log.*` 语句写入到通用内存缓冲区 `logcat`。从 Android 4.1（API 级别 16）开始，只能为特权系统应用授予读取 `logcat` 的权限（方法是声明 `READ_LOGS` 权限）。不过，Android 支持的设备极其多样，这些设备的预加载应用有时会声明 `READ_LOGS` 特权。因此，不建议直接将日志记录到 `logcat`，因为这更容易出现数据泄露。

确保在应用的非调试版本中对记录到 `logcat` 的所有日志进行清理。移除所有可能比较敏感的数据。加大防范措施，使用 R8 等工具移除“警告”和“错误”以外所有其他级别的日志。如果您需要更详细的日志，请使用内部存储空间并直接管理自己的日志，而不要使用系统日志。

## 影响

日志信息泄露这类漏洞的严重程度可能会因敏感数据的上下文和类型而异。总体而言，这类漏洞造成的影响就是会使个人身份信息和凭据等潜在关键信息失去机密性。

## 缓解措施

### 一般措施

根据[最小权限原则](https://www.cisa.gov/uscert/bsi/articles/knowledge/principles/least-privilege#:%7E:text=The%20Principle%20of%20Least%20Privilege%20states%20that%20a%20subject%20should,control%20the%20assignment%20of%20rights.)设定信任边界，这是在设计和实现期间采取的一般性预防措施。理想情况下，敏感数据不得跨越或超出任何信任区域。这增强了权限分离。

请勿记录敏感数据。尽可能只记录编译时常量。您可以使用 [ErrorProne 工具](https://errorprone.info/bugpattern/CompileTimeConstant)为编译时常量添加注解。

避免在日志中输出那些因触发特定错误而可能包含意外信息（包括敏感数据）的语句。在日志和错误日志中输出的数据应尽可能仅包含可预料的信息。

避免将日志记录到 `logcat`。这是因为，将日志记录到 `logcat` 可能会因应用具有 `READ_LOGS` 权限而造成隐私问题。此外，由于无法触发警报或无法查询，此类日志会变得无效。我们建议应用仅针对开发者 build 配置 `logcat` 后端。

大多数日志管理库都允许定义日志级别，以便为调试日志和生产日志记录不同数量的信息。在产品测试结束后，应立即更改日志级别，以便与“调试”级别区分开。

从生产环境中移除尽可能多的日志级别。如果您无法避免将日志保留在生产环境中，请从日志语句中移除非常量变量。可能会发生以下情况：

- 您可以从生产环境中移除所有日志。
- 您需要在生产环境中保留警告日志和错误日志。

对于这两种情况，请使用 R8 等库自动移除日志。如果尝试手动移除日志，则很容易出错。在代码优化过程中，可以将 R8 设置为安全地移除要保留用于调试、但要在生产环境中剥离的日志级别。

如果您要在生产环境中记录日志，请准备相关标志，以便在发生突发事件时有条件地关闭日志记录。在准备突发事件响应标志时应优先考虑以下事项：部署的安全性、部署的速度和简便性、隐去日志的彻底性、内存用量以及扫描每条日志消息的性能开销。

### 使用 R8 将日志从正式版 build 剥离到 logcat。

在 Android Studio 3.4 或 Android Gradle 插件 3.4.0 及更高版本中，R8 是用于优化和缩减代码的默认编译器。不过，您需要[启用 R8](https://developer.android.com/studio/build/shrink-code?hl=zh-cn#enable)。

R8 取代了 ProGuard，但项目根文件夹中的规则文件仍称为 `proguard-rules.pro`。以下代码段展示了一个 `proguard-rules.pro` 文件示例，它会从生产环境中移除警告和错误之外的所有日志：

```java
-assumenosideeffects class android.util.Log {
    private static final String TAG = "MyTAG";
    public static boolean isLoggable(java.lang.String, int);
    public static int v(TAG, "My log as verbose");
    public static int d(TAG, "My log as debug");
    public static int i(TAG, "My log as information");
}
```

以下 `proguard-rules.pro` 文件示例会从生产环境中移除所有日志：

```java
-assumenosideeffects class android.util.Log {
    private static final String TAG = "MyTAG";
    public static boolean isLoggable(java.lang.String, int);
    public static int v(TAG, "My log as verbose");
    public static int d(TAG, "My log as debug");
    public static int i(TAG, "My log as information");
    public static int w(TAG, "My log as warning");
    public static int e(TAG, "My log as error");
}
```

请注意，R8 提供应用缩减功能和日志剥离功能。如果您只想使用 R8 的日志剥离功能，请将以下代码添加到 `proguard-rules.pro` 文件中：

```java
-dontwarn **
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose

-optimizations !code/simplification/arithmetic,!code/allocation/variable
-keep class **
-keepclassmembers class *{*;}
-keepattributes *
```

### 对生产环境中包含敏感数据的所有最终日志进行清理

为避免泄露敏感数据，请确保在应用的非调试版本中对记录到 `logcat` 的所有日志进行清理。移除所有可能比较敏感的数据。

示例：

[Kotlin](https://developer.android.com/privacy-and-security/risks/log-info-disclosure?hl=zh-cn#kotlin)
```kotlin
data class Credential<T>(val data: String) {
  /** Returns a redacted value to avoid accidental inclusion in logs. */
  override fun toString() = "Credential XX"
}

fun checkNoMatches(list: List<Any>) {
    if (!list.isEmpty()) {
          Log.e(TAG, "Expected empty list, but was %s", list)
    }
}
```

[Java](https://developer.android.com/privacy-and-security/risks/log-info-disclosure?hl=zh-cn#java)
```java
public class Credential<T> {
  private T t;
  /** Returns a redacted value to avoid accidental inclusion in logs. */
  public String toString(){
         return "Credential XX";
  }
}

private void checkNoMatches(List<E> list) {
   if (!list.isEmpty()) {
          Log.e(TAG, "Expected empty list, but was %s", list);
   }
}
```

### 隐去日志中的敏感数据

如果您必须在日志中包含敏感数据，那么我们建议您先对日志进行清理，然后再输出这些日志，以移除或混淆处理敏感数据。为此，请使用以下方法之一：

- **令牌化** - 如果敏感数据存储在保险库（例如可以使用令牌引用密文的加密管理系统）中，请记录令牌，而不是敏感数据。
- **数据脱敏** - 数据脱敏是一种单向的不可逆过程。它会创建一个与原始敏感数据在结构上类似的版本，但会隐藏字段中包含的最敏感的信息。示例：将信用卡号码 `1234-5678-9012-3456` 替换为 `XXXX-XXXX-XXXX-1313`。在将您的应用发布为正式版之前，我们建议您先完成安全审核流程，以仔细审查数据脱敏的应用情况。 警告：如果仅发布部分敏感数据也会严重影响安全性（例如处理密码时），那么请不要使用数据脱敏。
- **隐去** - 隐去类似于脱敏，但会隐藏字段中包含的所有信息。示例：将信用卡号码 `1234-5678-9012-3456` 替换为 `XXXX-XXXX-XXXX-XXXX`。
- **过滤** - 在所选的日志记录库中采用格式字符串（如果尚未采用），以便于在日志语句中修改非常量值。

日志输出只能通过“日志清理程序”组件执行，该组件可确保所有日志在输出之前都经过清理，如以下代码段所示。

[Kotlin](https://developer.android.com/privacy-and-security/risks/log-info-disclosure?hl=zh-cn#kotlin)
```kotlin
data class ToMask<T>(private val data: T) {
  // Prevents accidental logging when an error is encountered.
  override fun toString() = "XX"

  // Makes it more difficult for developers to invoke sensitive data
  // and facilitates sensitive data usage tracking.
  fun getDataToMask(): T = data
}

data class Person(
  val email: ToMask<String>,
  val username: String
)

fun main() {
    val person = Person(
        ToMask("name@gmail.com"),
        "myname"
    )
    println(person)
    println(person.email.getDataToMask())
}
```
[Java](https://developer.android.com/privacy-and-security/risks/log-info-disclosure?hl=zh-cn#java)

```java
public class ToMask<T> {
  // Prevents accidental logging when an error is encountered.
  public String toString(){
         return "XX";
  }

  // Makes it more difficult for developers to invoke sensitive data
  // and facilitates sensitive data usage tracking.
  public T  getDataToMask() {
    return this;
  }
}

public class Person {
  private ToMask<String> email;
  private String username;

  public Person(ToMask<String> email, String username) {
    this.email = email;
    this.username = username;
  }
}

public static void main(String[] args) {
    Person person = new Person(
        ToMask("name@gmail.com"),
        "myname"
    );
    System.out.println(person);
    System.out.println(person.email.getDataToMask());
}
```

