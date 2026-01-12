---
title: Android-安全性-安全风险-不安全的broadcast-receive
date: 2025-12-1 08:11:07 +0800
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
description: 开发者不希望所有应用都调用有意导出的广播接收器，但未进行适当的访问权限控制，则该广播接收器可能会被滥用。
math: true
---
# 不安全的广播接收器

**OWASP 类别**： [MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

如果广播接收器实现不当，攻击者可能会发送恶意 intent，以便让易受攻击的应用执行不适用于外部调用方的操作。

此漏洞通常是指广播接收器无意中被导出的情形，这可能是通过在 AndroidManifest 中设置 [`android:exported="true"`](https://developer.android.com/guide/topics/manifest/receiver-element?hl=zh-cn#exported) 或通过以编程方式创建广播接收器（默认情况下会使接收器处于公开状态）而导致的。如果接收器不包含任何 intent 过滤器，则默认值为 `"false"`；如果接收器包含至少一个 intent 过滤器，则 android:exported 的默认值为 `"true"`。

如果开发者不希望所有应用都调用有意导出的广播接收器，但未进行适当的访问权限控制，则该广播接收器可能会被滥用。

## 影响

攻击者可能会滥用不安全实现的广播接收器，在未经授权的情况下访问应用中开发者不打算向第三方公开的行为。

## 缓解措施

### 完全避免此问题

如需彻底解决此问题，请将 `exported` 设置为 `false`：

```xml
<receiver android:name=".MyReceiver" android:exported="false">
    <intent-filter>
        <action android:name="com.example.myapp.MY_ACTION" />
    </intent-filter>
</receiver>
```

### 使用调用和回调

如果您出于内部应用用途（例如事件完成通知）使用广播接收器，则可以重构代码，以改为传递会在事件完成后触发的回调。

###### 事件完成监听器

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#kotlin)
```kotlin
interface EventCompletionListener {
    fun onEventComplete(data: String)
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#java)

```java
public interface EventCompletionListener {
    public void onEventComplete(String data);
}
```

###### 安全任务

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#kotlin)
```kotlin
class SecureTask(private val listener: EventCompletionListener?) {
    fun executeTask() {
        // Do some work...

        // Notify that the event is complete
        listener?.onEventComplete("Some secure data")
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#java)

```java
public class SecureTask {

    final private EventCompletionListener listener;

    public SecureTask(EventCompletionListener listener) {
        this.listener = listener;
    }

    public void executeTask() {
        // Do some work...

        // Notify that the event is complete
        if (listener != null) {
            listener.onEventComplete("Some secure data");
        }
    }
}
```

###### 主 Activity

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#kotlin)
```kotlin
class MainActivity : AppCompatActivity(), EventCompletionListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val secureTask = SecureTask(this)
        secureTask.executeTask()
    }

    override fun onEventComplete(data: String) {
        // Handle event completion securely
        // ...
    }
}
```

[Java](https://developer.android.com/privacy-and-security/risks/insecure-broadcast-receiver?hl=zh-cn#java)

```java
public class MainActivity extends AppCompatActivity implements EventCompletionListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        SecureTask secureTask = new SecureTask(this);
        secureTask.executeTask();
    }

    @Override
    public void onEventComplete(String data) {
        // Handle event completion securely
        // ...
    }
}
```

### 使用权限保护广播接收器

仅为[受保护的广播](https://developer.android.com/about/versions/12/reference/broadcast-intents-31?hl=zh-cn)（只有系统级应用才能发送的广播）或具有[自行声明的签名级权限](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)注册动态接收器。

## 资源

- [导出的接收器元素](https://developer.android.com/guide/topics/manifest/receiver-element?hl=zh-cn#exported)
- [“广播接收器权限”文档](https://developer.android.com/guide/components/broadcasts?hl=zh-cn#receiving-broadcasts-permissions)
- [受保护的广播 intent](https://developer.android.com/about/versions/12/reference/broadcast-intents-31?hl=zh-cn)

