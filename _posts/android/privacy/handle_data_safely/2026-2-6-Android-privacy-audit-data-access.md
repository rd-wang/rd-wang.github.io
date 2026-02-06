---
title: Android-隐私权-审核对数据的访问
date: 2026-2-6 15:19:56 +0800
categories:
  - Android
  - Privacy
tags:
  - Android
  - Privacy
description: 通过执行数据访问审核，您可以深入了解您的应用程序及其依赖项如何访问用户的私有数据。该流程适用于运行 Android 11（API 级别 30）及更高版本的设备，可帮助您更好地识别潜在的意外数据访问。
math: true
---
通过执行数据访问审核，您可以深入了解您的应用程序及其依赖项如何访问用户的私有数据。该流程适用于运行 Android 11（API 级别 30）及更高版本的设备，可帮助您更好地识别潜在的意外数据访问。您的应用可以注册 [`AppOpsManager.OnOpNotedCallback`](https://developer.android.com/reference/android/app/AppOpsManager.OnOpNotedCallback?hl=zh-cn) 实例，该实例可以在下列事件发生时执行操作：

- 应用的代码访问私密数据。为了帮助您确定应用的哪个逻辑部分调用了事件，您可以[按归属标签审核数据访问情况](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#audit-by-attribution-tag)。
- 依赖库或 SDK 中的代码访问私密数据。

数据访问审核在发生数据请求的线程上调用。这意味着如果应用中的第三方 SDK 或库调用了访问私人数据的 API，数据访问审核可以让 `OnOpNotedCallback` 检查关于调用的信息。通常，这个回调对象可以通过查看应用当前状态（例如当前线程的堆栈跟踪）来判断调用是来自你的应用还是 SDK。

## 记录数据访问

要使用 `AppOpsManager.OnOpNotedCallback` 实例执行数据访问审核，请在你打算审核数据访问的组件中实现回调逻辑，例如在一个 Activity 的 [`onCreate()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onCreate\(android.os.Bundle,%20android.os.PersistableBundle\)) 方法或应用程序的 [`onCreate()`](https://developer.android.com/reference/android/app/Application?hl=zh-cn#onCreate\(\)) 方法中。

> **注意：** 如果您的应用在多个组件（例如在前台服务和后台任务）中访问数据，请创建自定义[`Application`](https://developer.android.com/reference/kotlin/android/app/Application?hl=zh-cn)的子类，并在子类的  `onCreate()` 方法中定义 `AppOpsManager.OnOpNotedCallback`。

以下代码段定义了在单个 Activity 中用于审核数据访问的 `AppOpsManager.OnOpNotedCallback`：

[Kotlin](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#kotlin)
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    val appOpsCallback = object : AppOpsManager.OnOpNotedCallback() {
        private fun logPrivateDataAccess(opCode: String, trace: String) {
            Log.i(`MY_APP_TAG`, "Private data accessed. " +
                    "Operation: $opCode\nStack Trace:\n$trace")
        }

        override fun onNoted(syncNotedAppOp: SyncNotedAppOp) {
            logPrivateDataAccess(
                    syncNotedAppOp.op, Throwable().stackTrace.toString())
        }

        override fun onSelfNoted(syncNotedAppOp: SyncNotedAppOp) {
            logPrivateDataAccess(
                    syncNotedAppOp.op, Throwable().stackTrace.toString())
        }

        override fun onAsyncNoted(asyncNotedAppOp: AsyncNotedAppOp) {
            logPrivateDataAccess(asyncNotedAppOp.op, asyncNotedAppOp.message)
        }
    }

    val appOpsManager =
            getSystemService(AppOpsManager::class.java) as AppOpsManager
    appOpsManager.setOnOpNotedCallback(mainExecutor, appOpsCallback)
}

```

[Java](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#java)

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState,
        @Nullable PersistableBundle persistentState) {
    AppOpsManager.OnOpNotedCallback appOpsCallback =
            new AppOpsManager.OnOpNotedCallback() {
        private void logPrivateDataAccess(String opCode, String trace) {
            Log.i(MY_APP_TAG, "Private data accessed. " +
                    "Operation: $opCode\nStack Trace:\n$trace");
        }

        @Override
        public void onNoted(@NonNull SyncNotedAppOp syncNotedAppOp) {
            logPrivateDataAccess(syncNotedAppOp.getOp(),
                    Arrays.toString(new Throwable().getStackTrace()));
        }

        @Override
        public void onSelfNoted(@NonNull SyncNotedAppOp syncNotedAppOp) {
            logPrivateDataAccess(syncNotedAppOp.getOp(),
                    Arrays.toString(new Throwable().getStackTrace()));
        }

        @Override
        public void onAsyncNoted(@NonNull AsyncNotedAppOp asyncNotedAppOp) {
            logPrivateDataAccess(asyncNotedAppOp.getOp(),
                    asyncNotedAppOp.getMessage());
        }
    };

    AppOpsManager appOpsManager = getSystemService(AppOpsManager.class);
    if (appOpsManager != null) {
        appOpsManager.setOnOpNotedCallback(getMainExecutor(), appOpsCallback);
    }
}
```

在特定情况下，需要调用 `onAsyncNoted()` 和 `onSelfNoted()` 方法：

- 如果数据访问并非发生在应用 API 调用期间，则会调用 [`onAsyncNoted()`](https://developer.android.com/reference/android/app/AppOpsManager.OnOpNotedCallback?hl=zh-cn#onAsyncNoted\(android.app.AsyncNotedAppOp\)) 方法。最常见的例子是，应用注册了一个监听器，并且每次调用监听器的回调函数时都会进行数据访问。 传递到 `onAsyncNoted()` 中的 `AsyncNotedOp` 参数包含名为 `getMessage()` 的方法。此方法提供有关数据访问的更多信息。如果是位置信息回调，该消息将包含监听器的系统身份哈希值。
    
- [`onSelfNoted()`](https://developer.android.com/reference/android/app/AppOpsManager.OnOpNotedCallback?hl=zh-cn#onSelfNoted\(android.app.SyncNotedAppOp\))仅在极少数情况下，即当应用程序将自己的 UID 传递给 [`noteOp()`](https://developer.android.com/reference/android/app/AppOpsManager?hl=zh-cn#noteOp\(java.lang.String,%20int,%20java.lang.String,%20java.lang.String,%20java.lang.String\))时才会调用。 
    

## 通过归属标签审核数据访问

您的应用可能有几种主要用例，例如允许用户拍照并与联系人分享这些照片。如果您开发了一个多用途应用程序，则可以在审核其数据访问时，将归属标签应用于应用程序的每个部分。`attributionTag` 上下文会返回到传递给 [`onNoted()`](https://developer.android.com/reference/android/app/AppOpsManager.OnOpNotedCallback?hl=zh-cn#onNoted\(android.app.SyncNotedAppOp\))调用的对象中。这有助于您更轻松地将数据访问追溯到代码的逻辑部分。

要在您的应用中定义归属标签，请完成以下各节中的步骤。

### 在清单中声明归因标记

如果您的应用以 Android 12（API 级别 31）或更高版本为目标平台，您必须使用以下代码段中显示的格式在应用的清单文件中声明归属标记。如果您尝试使用未在应用的清单文件中声明的某个归属标记，则系统会为您创建一个 `null` 标记并在 Logcat 中记录一条消息。

```xml
<manifest ...>
    <!-- The value of "android:tag" must be a literal string, and the
         value of "android:label" must be a resource. The value of
         "android:label" is user-readable. -->
    <attribution android:tag="sharePhotos"
                 android:label="@string/share_photos_attribution_label" />
    ...
</manifest>
```

### 创建归属标记

如果您在某个 Activity 中访问数据（例如请求位置信息或访问用户的联系人列表），请在该 Activity 的 [`onCreate()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onCreate\(android.os.Bundle,%20android.os.PersistableBundle\)) 方法中调用 [`createAttributionContext()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#createAttributionContext\(java.lang.String\))，传入希望与应用某部分关联的归属标签。

以下代码段展示了如何为应用的 photo-location-sharing 部分创建归属标记：

[Kotlin](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#kotlin)
```kotlin
class SharePhotoLocationActivity : AppCompatActivity() {
    lateinit var attributionContext: Context

    override fun onCreate(savedInstanceState: Bundle?) {
        attributionContext = createAttributionContext("sharePhotos")
    }

    fun getLocation() {
        val locationManager = attributionContext.getSystemService(
                LocationManager::class.java) as LocationManager
        // Use "locationManager" to access device location information.
    }
}
```

[Java](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#java)
```java
public class SharePhotoLocationActivity extends AppCompatActivity {
    private Context attributionContext;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState,
            @Nullable PersistableBundle persistentState) {
        attributionContext = createAttributionContext("sharePhotos");
    }

    public void getLocation() {
        LocationManager locationManager =
                attributionContext.getSystemService(LocationManager.class);
        if (locationManager != null) {
            // Use "locationManager" to access device location information.
        }
    }
}
```


### 在访问日志中包含归属标记

更新 `AppOpsManager.OnOpNotedCallback` 回调函数，以便应用程序的日志包含您定义的归属标签的名称。

> **注意**：如果归属标记的返回值为 `null`，意味着当前的 [`Context`](https://developer.android.com/reference/android/content/Context?hl=zh-cn) 对象与您的应用程序的主体部分相关联。

以下代码段展示了记录归属代码的更新逻辑：

[Kotlin](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#kotlin)
```kotlin
val appOpsCallback = object : AppOpsManager.OnOpNotedCallback() {
    private fun logPrivateDataAccess(
            opCode: String, **attributionTag: String**, trace: String) {
        Log.i(MY_APP_TAG, "Private data accessed. " +
                    "Operation: $opCode\n " +
                    "Attribution Tag:$attributionTag\nStack Trace:\n$trace")
    }

    override fun onNoted(syncNotedAppOp: SyncNotedAppOp) {
        logPrivateDataAccess(syncNotedAppOp.op,
                **syncNotedAppOp.attributionTag**,
                Throwable().stackTrace.toString())
    }

    override fun onSelfNoted(syncNotedAppOp: SyncNotedAppOp) {
        logPrivateDataAccess(syncNotedAppOp.op,
                **syncNotedAppOp.attributionTag**,
                Throwable().stackTrace.toString())
    }

    override fun onAsyncNoted(asyncNotedAppOp: AsyncNotedAppOp) {
        logPrivateDataAccess(asyncNotedAppOp.op,
                **asyncNotedAppOp.attributionTag**,
                asyncNotedAppOp.message)
    }
}
```
[Java](https://developer.android.com/privacy-and-security/audit-data-access?hl=zh-cn#java)
```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState,
        @Nullable PersistableBundle persistentState) {
    AppOpsManager.OnOpNotedCallback appOpsCallback =
            new AppOpsManager.OnOpNotedCallback() {
        private void logPrivateDataAccess(String opCode,
                **String attributionTag**, String trace) {
            Log.i("MY_APP_TAG", "Private data accessed. " +
                    "Operation: $opCode\n " +
                    "Attribution Tag:$attributionTag\nStack Trace:\n$trace");
        }

        @Override
        public void onNoted(@NonNull SyncNotedAppOp syncNotedAppOp) {
            logPrivateDataAccess(syncNotedAppOp.getOp(),
                    **syncNotedAppOp.getAttributionTag()**,
                    Arrays.toString(new Throwable().getStackTrace()));
        }

        @Override
        public void onSelfNoted(@NonNull SyncNotedAppOp syncNotedAppOp) {
            logPrivateDataAccess(syncNotedAppOp.getOp(),
                    **syncNotedAppOp.getAttributionTag()**,
                    Arrays.toString(new Throwable().getStackTrace()));
        }

        @Override
        public void onAsyncNoted(@NonNull AsyncNotedAppOp asyncNotedAppOp) {
            logPrivateDataAccess(asyncNotedAppOp.getOp(),
                    **asyncNotedAppOp.getAttributionTag()**,
                    asyncNotedAppOp.getMessage());
        }
    };

    AppOpsManager appOpsManager = getSystemService(AppOpsManager.class);
    if (appOpsManager != null) {
        appOpsManager.setNotedAppOpsCollector(appOpsCollector);
    }
}
```
