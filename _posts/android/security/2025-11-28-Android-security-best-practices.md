---
title: Android-安全性-提高应用的安全性
date: 2025-11-28 09:52:09 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
description: 防止未经授权访问敏感信息，保护用户免受恶意软件和网络钓鱼攻击，并确保 Android 生态系统的整体完整性
math: true
---
# 提高应用的安全性

提高应用的安全性有助于维护用户信任和设备完整性。

本页介绍了若干最佳做法，可以极大地改善您的应用安全性。

## 强制采用安全通信方式

通过对您的应用与其他应用之间或您的应用与网站之间交换的数据采取保护措施，您可以提升应用的稳定性，并保护您发送和接收的数据。

### 保护应用之间的通信 

为了更安全地在应用之间进行通信，请将隐式 intent 与应用选择器、基于签名的权限以及不支持导出的 content provider 结合使用。

#### 显示应用选择器

如果隐式 intent 可以在用户设备上启动至少两个可能的应用，请明确显示应用选择器。此互动策略让用户可以向其信任的应用传输敏感信息。

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val intent = Intent(Intent.ACTION_SEND)
val possibleActivitiesList: List<ResolveInfo> =
        packageManager.queryIntentActivities(intent, PackageManager.MATCH_ALL)

// Verify that an activity in at least two apps on the user's device
// can handle the intent. Otherwise, start the intent only if an app
// on the user's device can handle the intent.
if (possibleActivitiesList.size > 1) {

    // Create intent to show chooser.
    // Title is something similar to "Share this photo with."

    val chooser = resources.getString(R.string.chooser_title).let { title ->
        Intent.createChooser(intent, title)
    }
    startActivity(chooser)
} else if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
}
```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
Intent intent = new Intent(Intent.ACTION_SEND);
List<ResolveInfo> possibleActivitiesList = getPackageManager()
        .queryIntentActivities(intent, PackageManager.MATCH_ALL);

// Verify that an activity in at least two apps on the user's device
// can handle the intent. Otherwise, start the intent only if an app
// on the user's device can handle the intent.
if (possibleActivitiesList.size() > 1) {

    // Create intent to show chooser.
    // Title is something similar to "Share this photo with."

    String title = getResources().getString(R.string.chooser_title);
    Intent chooser = Intent.createChooser(intent, title);
    startActivity(chooser);
} else if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}
```
**相关信息**：

- [显示应用选择器](https://developer.android.com/training/basics/intents/sending?hl=zh-cn#AppChooser)
- [Intent](https://developer.android.com/reference/android/content/Intent?hl=zh-cn)

#### 采用基于签名的权限

当您在受您控制或由您拥有的两个应用之间共享数据时，请使用_基于签名_的权限。此类权限不需要用户确认，而是检查访问数据的应用是否使用相同的签名密钥进行了签名。因此，这些权限能够提供更加顺畅、安全的用户体验。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <permission android:name="my_custom_permission_name"
                android:protectionLevel="signature" />
```

**相关信息**：

- [对您的应用进行签名](https://developer.android.com/studio/publish/app-signing?hl=zh-cn)
- [`android:protectionLevel`](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)

#### 禁止访问您的应用的 content provider

除非您打算从您的应用向不属于您的其他应用发送数据，否则请明确禁止其他开发者的应用访问您应用的 [`ContentProvider`](https://developer.android.com/reference/android/content/ContentProvider?hl=zh-cn) 对象。如果您的应用可在搭载 Android 4.1.1（API 级别 16）或更低版本的设备上安装，此设置尤为重要，因为在这些版本的 Android 中，[`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn) 元素的 [`android:exported`](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn#exported) 属性默认设为 `true`。
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application ... >
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            ...
            android:exported="false">
            <!-- Place child elements of <provider> here. -->
        </provider>
        ...
    </application>
</manifest>
```


### 在显示敏感信息前索要凭据

如果用户需要提供凭据才可以访问应用中的敏感信息或付费内容时，请要求其提供 PIN 码/密码/图案或生物识别凭据（如人脸识别或指纹识别）。

如需详细了解如何索要生物识别凭据，请参阅[生物识别身份验证指南](https://developer.android.com/training/sign-in/biometric-auth?hl=zh-cn)。

### 应用网络安全措施

下面几个部分将介绍如何提升应用的网络安全性。

#### 使用 TLS 流量

如果您的应用与某个网络服务器通信，而该服务器具有由公认的可信证书授权机构 (CA) 颁发的证书，请使用如下所示的 HTTPS 请求：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val url = URL("https://www.google.com")
val urlConnection = url.openConnection() as HttpsURLConnection
urlConnection.connect()
urlConnection.inputStream.use {
    ...
}
```


[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
URL url = new URL("https://www.google.com");
HttpsURLConnection urlConnection = (HttpsURLConnection) url.openConnection();
urlConnection.connect();
InputStream in = urlConnection.getInputStream();
```

#### 添加网络安全配置

如果您的应用使用新的或自定义的 CA，您可以在配置文件中声明您的网络安全设置。通过这种方式，您可以在不修改任何应用代码的前提下创建配置。

如需向应用添加网络安全配置文件，请按以下步骤操作：

1. 在应用的清单中声明配置：
	```xml
	<manifest ... >
	    <application
	        android:networkSecurityConfig="@xml/network_security_config"
	        ... >
	        <!-- Place child elements of <application> element here. -->
	    </application>
	</manifest>
	```


5. 添加 XML 资源文件，位于 `res/xml/network_security_config.xml`。
    
    通过禁用明文流量，指定流向特定网域的所有流量都必须使用 HTTPS：
    ```xml
    <network-security-config>
        <domain-config cleartextTrafficPermitted="false">
            <domain includeSubdomains="true">secure.example.com</domain>
            ...
        </domain-config>
    </network-security-config>
     ```
    在开发过程中，您可以使用 `<debug-overrides>` 元素来明确允许用户安装的证书。此元素可在调试和测试期间替换应用的关键安全选项，而不会影响应用的版本配置。以下代码段展示了如何在应用的网络安全配置 XML 文件中定义此元素：
	```xml
	<network-security-config>
		<debug-overrides>
			<trust-anchors>
				<certificates src="user" />
			</trust-anchors>
		</debug-overrides>
	</network-security-config>
	```

**相关信息**：[网络安全配置](https://developer.android.com/training/articles/security-config?hl=zh-cn)

#### 创建您自己的信任管理器

您的 TLS 检查工具不应接受所有证书。如果您的用例符合以下情形之一，您可能需要设置信任管理器来处理收到的所有 TLS 警告：

- 您与之通信的网络服务器具有由新 CA 或自定义 CA 签名的证书。
- 您所使用的设备不信任该 CA。
- 您无法使用[网络安全配置](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#network-security-config)。

如需详细了解如何完成这些步骤，请参阅有关处理[未知证书授权机构](https://developer.android.com/training/articles/security-ssl?hl=zh-cn#UnknownCa)的讨论。

**相关信息**：

- [通过网络协议确保安全](https://developer.android.com/training/articles/security-ssl.html?hl=zh-cn)
- [`CertificateFactory`](https://developer.android.com/reference/java/security/cert/CertificateFactory?hl=zh-cn)
- [`HttpsURLConnection`](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn)
- [`TrustManager`](https://developer.android.com/reference/javax/net/ssl/TrustManager?hl=zh-cn)

### 谨慎使用 WebView 对象

应用中的 [`WebView`](https://developer.android.com/reference/android/webkit/WebView?hl=zh-cn) 对象不应允许用户转到超出您控制范围的网站。请尽可能使用许可名单来限制应用的 `WebView` 对象加载的内容。

此外，除非您可以完全控制并信任应用的 `WebView` 对象中的内容，否则绝不要启用 [JavaScript 接口支持](https://developer.android.com/guide/webapps/webview?hl=zh-cn#UsingJavaScript)。

#### 使用 HTML 消息通道

如果应用必须在搭载 Android 6.0（API 级别 23）及更高版本的设备上使用 JavaScript 接口支持，请改为使用 HTML 消息通道在网站与应用之间进行通信，如以下代码段中所示：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val myWebView: WebView = findViewById(R.id.webview)

// channel[0] and channel[1] represent the two ports.
// They are already entangled with each other and have been started.
val channel: Array<out WebMessagePort> = myWebView.createWebMessageChannel()

// Create handler for channel[0] to receive messages.
channel[0].setWebMessageCallback(object : WebMessagePort.WebMessageCallback() {

    override fun onMessage(port: WebMessagePort, message: WebMessage) {
        Log.d(TAG, "On port $port, received this message: $message")
    }
})

// Send a message from channel[1] to channel[0].
channel[1].postMessage(WebMessage("My secure message"))
```

[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
WebView myWebView = (WebView) findViewById(R.id.webview);

// channel[0] and channel[1] represent the two ports.
// They are already entangled with each other and have been started.
WebMessagePort[] channel = myWebView.createWebMessageChannel();

// Create handler for channel[0] to receive messages.
channel[0].setWebMessageCallback(new WebMessagePort.WebMessageCallback() {
    @Override
    public void onMessage(WebMessagePort port, WebMessage message) {
         Log.d(TAG, "On port " + port + ", received this message: " + message);
    }
});

// Send a message from channel[1] to channel[0].
channel[1].postMessage(new WebMessage("My secure message"));
```
**相关信息**：

- [`WebMessage`](https://developer.android.com/reference/android/webkit/WebMessage?hl=zh-cn)
- [`WebMessagePort`](https://developer.android.com/reference/android/webkit/WebMessagePort?hl=zh-cn)


## 提供恰当的权限

仅请求应用正常运行所需的最低数量的权限。请尽可能放弃应用不再需要的权限。

### 使用 intent 转移权限

如果某项操作可以在其他应用中完成，应尽量避免通过在您的应用中添加权限来完成此操作，而应使用 intent 将请求转给已具有相应权限的其他应用。

以下示例展示了如何使用 intent 将用户跳转到“通讯录”应用，而不是请求 [`READ_CONTACTS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#READ_CONTACTS) 和 [`WRITE_CONTACTS`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_CONTACTS) 权限：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
// Delegates the responsibility of creating the contact to a contacts app,
// which has already been granted the appropriate WRITE_CONTACTS permission.
Intent(Intent.ACTION_INSERT).apply {
    type = ContactsContract.Contacts.CONTENT_TYPE
}.also { intent ->
    // Make sure that the user has a contacts app installed on their device.
    intent.resolveActivity(packageManager)?.run {
        startActivity(intent)
    }
}
```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
// Delegates the responsibility of creating the contact to a contacts app,
// which has already been granted the appropriate WRITE_CONTACTS permission.
Intent insertContactIntent = new Intent(Intent.ACTION_INSERT);
insertContactIntent.setType(ContactsContract.Contacts.CONTENT_TYPE);

// Make sure that the user has a contacts app installed on their device.
if (insertContactIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(insertContactIntent);
}
```
此外，如果您的应用需要执行基于文件的 I/O 操作（如访问存储空间或选择文件），它不需要具备特殊的权限，因为系统可以代替您的应用完成这些操作。更好的一点是，在用户选择位于特定 URI 的内容后，发出调用的应用就可以获得所选资源的相关权限。

**相关信息**：

- [常用 intent](https://developer.android.com/guide/components/intents-common?hl=zh-cn)
- [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn)

### 在应用之间安全地共享数据

遵循以下最佳实践，以更安全的方式与其他应用共享您应用的内容：

- 根据需要强制实施只读或只写权限。
- 使用 [`FLAG_GRANT_READ_URI_PERMISSION`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#FLAG_GRANT_READ_URI_PERMISSION) 和 [`FLAG_GRANT_WRITE_URI_PERMISSION`](https://developer.android.com/reference/android/content/Intent?hl=zh-cn#FLAG_GRANT_WRITE_URI_PERMISSION) 标志，为客户端提供对数据的一次性访问权限。
- 在共享数据时，使用 `content://` URI，而不要使用 `file://` URI。[`FileProvider`](https://developer.android.com/reference/androidx/core/content/FileProvider?hl=zh-cn) 的实例会为您执行这一操作。

以下代码段展示了如何使用 URI 权限授予标志和 content provider 权限，在独立的 PDF 查看器应用中显示应用的 PDF 文件：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)

```kotlin
// Create an Intent to launch a PDF viewer for a file owned by this app.
Intent(Intent.ACTION_VIEW).apply {
    data = Uri.parse("content://com.example/personal-info.pdf")

    // This flag gives the started app read access to the file.
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}.also { intent ->
    // Make sure that the user has a PDF viewer app installed on their device.
    intent.resolveActivity(packageManager)?.run {
        startActivity(intent)
    }
}
```

[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
// Create an Intent to launch a PDF viewer for a file owned by this app.
Intent viewPdfIntent = new Intent(Intent.ACTION_VIEW);
viewPdfIntent.setData(Uri.parse("**content://**com.example/personal-info.pdf"));

// This flag gives the started app read access to the file.
viewPdfIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);

// Make sure that the user has a PDF viewer app installed on their device.
if (viewPdfIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(viewPdfIntent);
}
```

>**注意**：从可写的应用主目录执行文件的行为[违反了 W^X 安全机制](https://en.wikipedia.org/wiki/W%5EX)。因此，如果不可信应用以 Android 10（API 级别 29）及更高版本为目标平台，则其无法对应用主目录中的文件调用 `exec()`，而只能调用在应用的 APK 文件中嵌入的二进制代码。此外，以 Android 10 及更高版本为目标平台的应用也无法在内存中修改通过 `dlopen()` 打开的文件中的可执行代码。其中包括含有文本重定位的所有共享对象 (`.so`) 文件。

**相关信息**：
[`android:grantUriPermissions`](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn#gprmsn)

## 安全地存储数据

尽管您的应用可能需要访问用户的敏感信息，但用户只有在相信您会妥善保护数据的情况下，才会授予您的应用访问这些数据的权限。

### 将私有数据存储在内部存储设备中

将所有私有用户数据存储在设备的内部存储设备中，该存储设备已根据应用进行了沙盒化处理。您的应用不需要请求权限即可查看这些文件，而其他应用则无法访问这些文件。作为一项额外的安全措施，当用户卸载应用时，设备会删除应用保存在内部存储空间中的所有文件。

以下代码段展示了一种将数据写入内部存储空间的方法：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
// Creates a file with this name, or replaces an existing file
// that has the same name. Note that the file name cannot contain
// path separators.
val FILE_NAME = "sensitive_info.txt"
val fileContents = "This is some top-secret information!"
File(filesDir, FILE_NAME).bufferedWriter().use { writer ->
    writer.write(fileContents)
}
```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
// Creates a file with this name, or replaces an existing file
// that has the same name. Note that the file name cannot contain
// path separators.
final String FILE_NAME = "sensitive_info.txt";
String fileContents = "This is some top-secret information!";
try (BufferedWriter writer =
             new BufferedWriter(new FileWriter(new File(getFilesDir(), FILE_NAME)))) {
    writer.write(fileContents);
} catch (IOException e) {
    // Handle exception.
}
```

以下代码段展示了相反的操作，即从内部存储空间读取数据：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val FILE_NAME = "sensitive_info.txt"
val contents = File(filesDir, FILE_NAME).bufferedReader().useLines { lines ->
    lines.fold("") { working, line ->
        "$working\n$line"
    }
}
```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)

```java
final String FILE_NAME = "sensitive_info.txt";
StringBuffer stringBuffer = new StringBuffer();
try (BufferedReader reader =
             new BufferedReader(new FileReader(new File(getFilesDir(), FILE_NAME)))) {

    String line = reader.readLine();
    while (line != null) {
        stringBuffer.append(line).append('\n');
        line = reader.readLine();
    }
} catch (IOException e) {
    // Handle exception.
}
```

**相关信息**：

- [数据和文件存储概览](https://developer.android.com/guide/topics/data/data-storage?hl=zh-cn)
- [`FileInputStream`](https://developer.android.com/reference/java/io/FileInputStream?hl=zh-cn)
- [`FileOutputStream`](https://developer.android.com/reference/java/io/FileOutputStream?hl=zh-cn)
- [`Context.MODE_PRIVATE`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#MODE_PRIVATE)

### 根据用例将数据存储在外部存储设备中

使用外部存储设备存储应用专属的大型非敏感文件，以及应用与其他应用共享的文件。所使用的具体 API 取决于您的应用是访问应用专属文件，还是访问共享文件。

如果文件不包含私密信息或敏感信息，而仅在您的应用内对用户有价值，请将该文件存储在[外部存储设备上的应用专属目录](https://developer.android.com/training/data-storage/app-specific?hl=zh-cn#external)中。

如果您的应用需要访问或存储对其他应用有价值的文件，请根据您的用例使用以下某个 API：

- **媒体文件**：如需存储和访问在应用之间共享的图片、音频文件和视频，请[使用 Media Store API](https://developer.android.com/training/data-storage/shared/media?hl=zh-cn)。
- **其他文件**：如需存储和访问其他类型的共享文件（包括已下载的文件），请[使用存储访问框架](https://developer.android.com/training/data-storage/shared/documents-files?hl=zh-cn)。

#### 检查存储卷的可用性

如果您的应用与可移动外部存储设备交互，请注意，在应用尝试访问该存储设备时，用户可能移除了该设备。请添加逻辑以[验证存储设备是否可用](https://developer.android.com/training/data-storage/app-specific?hl=zh-cn#external-verify-availability)。

#### 检查数据有效性

如果您的应用使用外部存储设备中的数据，请确保数据的内容没有遭到损坏或修改。添加逻辑来处理不再采用稳定格式的文件。

以下代码段举例说明了哈希验证程序：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val hash = calculateHash(stream)
// Store "expectedHash" in a secure location.
if (hash == expectedHash) {
    // Work with the content.
}

// Calculating the hash code can take quite a bit of time, so it shouldn't
// be done on the main thread.
suspend fun calculateHash(stream: InputStream): String {
    return withContext(Dispatchers.IO) {
        val digest = MessageDigest.getInstance("SHA-512")
        val digestStream = DigestInputStream(stream, digest)
        while (digestStream.read() != -1) {
            // The DigestInputStream does the work; nothing for us to do.
        }
        digest.digest().joinToString(":") { "%02x".format(it) }
    }
}
```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
Executor threadPoolExecutor = Executors.newFixedThreadPool(4);
private interface HashCallback {
    void onHashCalculated(@Nullable String hash);
}

boolean hashRunning = calculateHash(inputStream, threadPoolExecutor, hash -> {
    if (Objects.equals(hash, expectedHash)) {
        // Work with the content.
    }
});

if (!hashRunning) {
    // There was an error setting up the hash function.
}

private boolean calculateHash(@NonNull InputStream stream,
                              @NonNull Executor executor,
                              @NonNull HashCallback hashCallback) {
    final MessageDigest digest;
    try {
        digest = MessageDigest.getInstance("SHA-512");
    } catch (NoSuchAlgorithmException nsa) {
        return false;
    }

    // Calculating the hash code can take quite a bit of time, so it shouldn't
    // be done on the main thread.
    executor.execute(() -> {
        String hash;
        try (DigestInputStream digestStream =
                new DigestInputStream(stream, digest)) {
            while (digestStream.read() != -1) {
                // The DigestInputStream does the work; nothing for us to do.
            }
            StringBuilder builder = new StringBuilder();
            for (byte aByte : digest.digest()) {
                builder.append(String.format("%02x", aByte)).append(':');
            }
            hash = builder.substring(0, builder.length() - 1);
        } catch (IOException e) {
            hash = null;
        }

        final String calculatedHash = hash;
        runOnUiThread(() -> hashCallback.onHashCalculated(calculatedHash));
    });
    return true;
}
```
### 仅将非敏感数据存储在缓存文件中

若要加快对非敏感应用数据的访问，您可以将其存储在设备的缓存中。对于超过 1 MB 的缓存，请使用 [`getExternalCacheDir()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getExternalCacheDir\(\))；对于未超过 1 MB 的缓存，请使用 [`getCacheDir()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getCacheDir\(\))。这两种方法都会为您提供 [`File`](https://developer.android.com/reference/java/io/File?hl=zh-cn) 对象，其中包含应用的缓存数据。

以下代码段展示了如何缓存应用最近下载的文件：

[Kotlin](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#kotlin)
```kotlin
val cacheFile = File(myDownloadedFileUri).let { fileToCache ->
    File(cacheDir.path, fileToCache.name)
}

```
[Java](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#java)
```java
File cacheDir = getCacheDir();
File fileToCache = new File(myDownloadedFileUri);
String fileToCacheName = fileToCache.getName();
File cacheFile = new File(cacheDir.getPath(), fileToCacheName);
```

>**注意**：如果您使用 [`getExternalCacheDir()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getExternalCacheDir\(\)) 将应用缓存放在共享的存储空间中，用户可能会在应用运行期间弹出包含此存储空间的介质。添加相应逻辑以妥善处理此用户行为导致的缓存未命中。

>**注意**：这些文件没有实施任何强制的安全措施。因此，任何以 Android 10（API 级别 29）或更低版本为目标平台且拥有 [`WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#WRITE_EXTERNAL_STORAGE) 权限的应用均可访问此缓存中的内容。

**相关信息**：[数据和文件存储概览](https://developer.android.com/guide/topics/data/data-storage?hl=zh-cn)

### 在隐私模式下使用 SharedPreferences

在使用 [`getSharedPreferences()`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#getSharedPreferences\(java.lang.String,%20int\)) 创建或访问您的应用的 [`SharedPreferences`](https://developer.android.com/reference/android/content/SharedPreferences?hl=zh-cn) 对象时，请使用 [`MODE_PRIVATE`](https://developer.android.com/reference/android/content/Context?hl=zh-cn#MODE_PRIVATE)。如此一来，只有您的应用才能访问共享偏好设置文件中的信息。

如果想要在应用之间共享数据，请勿使用 `SharedPreferences` 对象，而应按照相关步骤操作，[在应用之间安全地共享数据](https://developer.android.com/privacy-and-security/security-best-practices?hl=zh-cn#permissions-share-data)。

Security 库还提供了 [EncryptedSharedPreferences](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences?hl=zh-cn) 类，可用于封装 SharedPreferences 类并自动加密键和值。

**相关信息**：

- [数据和文件存储概览](https://developer.android.com/guide/topics/data/data-storage?hl=zh-cn)
- [Security 库中包含的类](https://developer.android.com/topic/security/data?hl=zh-cn#classes-in-library)

## 确保服务和依赖项处于最新状态 

大多数应用使用外部库和设备系统信息来完成特定的任务。通过及时更新应用的依赖项，您可以提高这些通信点的安全性。

### 检查 Google Play 服务安全提供程序

>**注意**：本部分仅适用于针对安装了 [Google Play 服务](https://developers.google.com/android/guides/overview?hl=zh-cn)的设备而设计的应用。

如果您的应用使用 Google Play 服务，请确保在安装了您的应用的设备上，该服务处于最新状态。在界面线程之外异步执行检查。如果该服务在设备上并未处于最新版本，则会触发授权错误。

如需确定装有您应用的设备上的 Google Play 服务是否处于最新版本，请按照[更新安全提供程序以防范 SSL 攻击](https://developer.android.com/training/articles/security-gms-provider?hl=zh-cn)指南中的步骤操作。

**相关信息**：

- [`ProviderInstaller`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller?hl=zh-cn)
- [`ProviderInstaller.ProviderInstallListener`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener?hl=zh-cn)

### 更新所有应用依赖项

在部署您的应用前，请确保所有库、SDK 和其他依赖项都处于最新状态：

- 对于 Android SDK 等第一方依赖项，请使用 Android Studio 中提供的更新工具，如 [SDK 管理器](https://developer.android.com/studio/intro/update?hl=zh-cn#sdk-manager)。
- 对于第三方依赖项，请检查您的应用所用库的网站，并安装所有可用的更新和安全补丁。

**相关信息**：[添加 build 依赖项](https://developer.android.com/studio/build/dependencies?hl=zh-cn#google_and_android_support_repositories)

## 更多信息

如需详细了解如何提高应用的安全性，请查看以下资源：

- [核心应用质量安全核对清单](https://developer.android.com/distribute/essentials/quality/core?hl=zh-cn#sc)
- [应用安全性改进计划](https://developer.android.com/google/play/asi?hl=zh-cn)
- [YouTube 上的 Android Developers 频道](https://www.youtube.com/user/androiddevelopers?hl=zh-cn)
- [Android 网络安全配置 Codelab](https://developer.android.com/codelabs/android-network-security-config?hl=zh-cn#0)
- [Android Protected Confirmation：进一步提高交易安全性](https://android-developers.googleblog.com/2018/10/android-protected-confirmation.html)
