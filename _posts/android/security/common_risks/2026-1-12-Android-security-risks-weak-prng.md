---
title: Android-安全性-安全风险-弱 PRNG
date: 2026-1-12 11:05:17 +0800
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
description: 如果在身份验证等安全环境中使用非加密安全的PRNG，攻击者可能能够猜到随机生成的数字，并获得对特权数据或功能的访问权限。
math: true
---
**OWASP 类别：**[MASVS-CRYPTO：加密](https://mas.owasp.org/MASVS/06-MASVS-CRYPTO)

## 概览

伪随机数生成器 (PRNG) 是一种根据称为种子的起始值生成可预测的数字序列的算法。由 PRNG 生成的数字序列在属性上与真正的随机数序列**大致**相同，但前者创建速度更快，计算开销也更低。

也就是说，对于模拟真正的随机数序列的熵分配，在均匀性方面，PRNG 比弱 RNG（例如 `java.math.Random`）更有保证。真正随机数的生成需要专门的设备，通常不在正常开发的范围之内。本文并不涵盖真正随机数的生成，仅重点介绍 PRNG，因为 PRNG 是当下使用的标准方法。

如果开发者使用常规 PRNG 进行加密，而不采用确保加密安全的 PRNG (CSPRNG)，就会出现弱 PRNG 漏洞。CSPRNG 有更严格的要求，当种子未知时，它们只能让攻击者在区分输出序列与实际随机序列方面占据微乎其微的优势。

当系统使用可预测的种子（例如开发者硬编码的种子）来初始化 PRNG 或 CSPRNG 时，攻击者或许还可以猜测生成的种子序列，因为攻击者可以猜测出这些种子，从而预测 PRNG 生成的输出。

## 影响

如果在身份验证等安全环境中使用非加密安全的PRNG，攻击者可能能够猜到随机生成的数字，并获得对特权数据或功能的访问权限。

## 缓解措施

### 常规

- 如果会有安全方面的影响，请使用 [`java.security.SecureRandom`](https://developer.android.com/reference/java/security/SecureRandom?hl=zh-cn)。
- 对于所有其他情况，请使用 [`java.util.Random`](https://developer.android.com/reference/java/util/Random?hl=zh-cn)
- **切勿**使用 [`Math.random`](https://developer.android.com/reference/java/lang/Math?hl=zh-cn#random\(\))！

### java.security.SecureRandom

用于安全用途时，**建议**使用。如果 Linux 内核版本为 5.17 或以上，或者可以阻止线程，可等待累积足够多的熵后再生成随机数（即使用 `/dev/random`）。为此，请调用 `getInstanceStrong()`：

[Kotlin](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#kotlin)
```kotlin
val rand = SecureRandom.getInstanceStrong()
```
[Java](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#java)

```java
SecureRandom rand = SecureRandom.getInstanceStrong();
```

但是在 5.17 之前的 Linux 内核版本中，如果在生成随机数时不可阻止线程，那么应直接调用 `SecureRandom` 构造函数：

[Kotlin](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#kotlin)
```kotlin
import java.security.SecureRandom

object generateRandom {
    @JvmStatic
    fun main(args: Array<String>) {
        // Create instance of SecureRandom class
        val rand = SecureRandom()

        // Generate random integers in range 0 to 999
        val rand_int = rand.nextInt(1000)

        // Use rand_int for security & authentication
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#java)

```java
import java.security.SecureRandom;

public class generateRandom {

    public static void main(String args[])
    {
        // Create instance of SecureRandom class
        SecureRandom rand = new SecureRandom();

        // Generate random integers in range 0 to 999
        int rand_int = rand.nextInt(1000);

        // Use rand_int for security & authentication
    }
}
```

`SecureRandom` 会从 `/dev/urandom` 获取默认种子，并且系统在构建或获取对象时会自动使用它，因此无需明确获取 PRNG 的种子。一般来说，不建议对 SecureRandom 进行任何确定性的使用（如果这样做会导致对种子值进行硬编码，而任何人在解压缩该应用时都能看到该种子值，便更为如此）。如果开发者想要生成可重现的伪随机输出，应使用更合适的基元，例如 HMAC、HKDF、SHAKE 等。

### java.util.Random

**避免**用于安全/身份验证目的，适用于任何其他用途。

[Kotlin](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#kotlin)
```kotlin
import java.util.Random

object generateRandom {
    @JvmStatic
    fun main(args: Array<String>) {
        // Create instance of SecureRandom class
        val rand = Random()

        // Generate random integers in range 0 to 999
        val rand_int = rand.nextInt(1000)
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/weak-prng?hl=zh-cn#java)

```java
import java.util.Random;

public class generateRandom {

    public static void main(String args[])
    {
        // Create instance of Random class
        Random rand = new Random();

        // Generate random integers in range 0 to 999
        int rand_int = rand.nextInt(1000);
    }
}
```

## 资源

- [java.security.SecureRandom](https://developer.android.com/reference/java/security/SecureRandom?hl=zh-cn)
    
- [java.util.Random](https://developer.android.com/reference/java/util/Random?hl=zh-cn)
    
- [Math.random](https://developer.android.com/reference/java/lang/Math?hl=zh-cn#random\(\))
    
- [可预测的种子 CWE](https://cwe.mitre.org/data/definitions/337.html)
    
- [弱加密 PRNG CWE](https://cwe.mitre.org/data/definitions/338.html)
    
- [Java Secure Random](https://www.baeldung.com/java-secure-random)
    
- [Java Random 与 SecureRandom](https://www.geeksforgeeks.org/random-vs-secure-random-numbers-java/)
    
- [如何使用 SecureRandom](https://tersesystems.com/blog/2015/12/17/the-right-way-to-use-securerandom/)
    
- [Python PRNG 安全指南](https://rules.sonarsource.com/python/RSPEC-2245)
    
- [OWASP 加密存储备忘单](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation)
    
- [CVE-2013-6386：Drupal 中的弱 PRNG 漏洞](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2013-6386)
    
- [CVE-2006-3419：Tor 中的弱 PRNG 漏洞](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-3419)
    
- [CVE-2008-4102：Joomla 中的可预测种子](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4102)
    
- [Linux 内核随机补丁](https://lwn.net/Articles/884875/)

