---
title: Android-安全性-安全风险-内容解析器
date: 2025-11-28 14:54:08 +0800
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
description: 用于为应用提供内容模型访问权限，可能会导致应用的受保护数据遭到未经授权方的窃取或篡改。
math: true
---


# 内容解析器

**OWASP 类别**：[MASVS-PLATFORM：平台互动](https://mas.owasp.org/MASVS/09-MASVS-PLATFORM)

## 概览

根据相关[文档](https://developer.android.com/reference/android/content/ContentResolver?hl=zh-cn)，`ContentResolver` 是一种“用于为应用提供内容模型访问权限的类”。ContentResolver 可以提供与来自以下各项的内容进行互动以及提取或修改这些内容时使用的方法：

- 安装式应用 (`content://` URI scheme)
- 文件系统 (`file://` URI scheme)
- Android 提供的辅助性 API (`android.resource://` URI scheme)。

总而言之，与 `ContentResolver` 相关的漏洞属于[混淆代理](https://en.wikipedia.org/wiki/Confused_deputy_problem)类，因为攻击者可以利用存在漏洞的应用的特权来访问受保护的内容。

## 风险：基于不受信任的 file:// URI 的滥用行为

通过 `file://` URI 漏洞滥用 `ContentResolver` 是指利用 `ContentResolver` 返回 URI 所述的文件描述符的功能。此漏洞会影响 `ContentResolver` [API](https://developer.android.com/reference/android/content/ContentResolver?hl=zh-cn) 的 `openFile()`、`openFileDescriptor()`、`openInputStream()`、`openOutputStream()` 或 `openAssetFileDescriptor()` 等函数。攻击者可能会通过由攻击者完全或部分控制的 `file://` URI 来滥用此漏洞，以强制应用访问原本不可访问的文件，例如内部数据库或共享的偏好设置。

一种可能的攻击情景是：创建一个恶意的图库或文件选择器，当存在漏洞的应用使用它时，它便会返回恶意 URI。

下面列出了此攻击的一些变体：

- 由攻击者完全控制且指向应用内部文件的 `file://` URI
- `file://` URI 的一部分由攻击者控制，因此容易遭到路径遍历攻击
- 以由攻击者控制且指向应用内部文件的符号链接为目标的 `file://` URI
- 与上述变体类似，但在这种情况下，攻击者会反复将符号链接的目标从合法目标切换为应用的内部文件。其目的是利用可能进行的安全检查与文件路径使用之间的竞态条件

### 影响

利用此漏洞造成的影响因 ContentResolver 的用途而异。在很多情况下，这可能会导致应用的受保护数据遭到未经授权方的窃取或篡改。

### 缓解措施

如需缓解此漏洞，请使用以下算法验证相应文件描述符。通过验证后，您就可以放心使用该文件描述符了。

**注意**：只有该文件描述符是安全的。如果您决定重复使用此 URI，则需要重复验证流程。

[Kotlin](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#kotlin)
```kotlin
fun isValidFile(ctx: Context, pfd: ParcelFileDescriptor, fileUri: Uri): Boolean {
    // Canonicalize to resolve symlinks and path traversals.
    val fdCanonical = File(fileUri.path!!).canonicalPath

    val pfdStat: StructStat = Os.fstat(pfd.fileDescriptor)

    // Lstat doesn't follow the symlink.
    val canonicalFileStat: StructStat = Os.lstat(fdCanonical)

    // Since we canonicalized (followed the links) the path already,
    // the path shouldn't point to symlink unless it was changed in the
    // meantime.
    if (OsConstants.S_ISLNK(canonicalFileStat.st_mode)) {
        return false
    }

    val sameFile =
        pfdStat.st_dev == canonicalFileStat.st_dev &&
        pfdStat.st_ino == canonicalFileStat.st_ino

    if (!sameFile) {
        return false
    }

    return !isBlockedPath(ctx, fdCanonical)
}

fun isBlockedPath(ctx: Context, fdCanonical: String): Boolean {
    // Paths that should rarely be exposed
    if (fdCanonical.startsWith("/proc/") ||
        fdCanonical.startsWith("/data/misc/")) {
        return true
    }

    // Implement logic to block desired directories. For example, specify
    // the entire app data/ directory to block all access.
}
```
[Java](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#java)
```java
boolean isValidFile(Context ctx, ParcelFileDescriptor pfd, Uri fileUri) {
    // Canonicalize to resolve symlinks and path traversals
    String fdCanonical = new File(fileUri.getPath()).getCanonicalPath();

    StructStat pfdStat = Os.fstat(pfd.getFileDescriptor());

    // Lstat doesn't follow the symlink.
    StructStat canonicalFileStat = Os.lstat(fdCanonical);

    // Since we canonicalized (followed the links) the path already,
    // the path shouldn't point to symlink unless it was changed in the meantime
    if (OsConstants.S_ISLNK(canonicalFileStat.st_mode)) {
        return false;
    }

    boolean sameFile =
        pfdStat.stDev == canonicalFileStat.stDev && pfdStat.stIno == canonicalFileStat.stIno;

    if (!sameFile) {
        return false;
    }

    return !isBlockedPath(ctx, fdCanonical);
}

boolean isBlockedPath(Context ctx, String fdCanonical) {
       
        // Paths that should rarely be exposed
        if (fdCanonical.startsWith("/proc/") || fdCanonical.startsWith("/data/misc/")) {
            return true;
        }

        // Implement logic to block desired directories. For example, specify
        // the entire app data/ directory to block all access.
}
```
  

---

## 风险：基于不受信任的 content:// URI 的滥用行为

当将由攻击者完全或部分控制的 URI 传递到 `ContentResolver` API，以对原本不可访问的内容执行操作时，便会发生通过 `content://` URI 漏洞滥用 `ContentResolver` 的攻击。

这种攻击主要有两种情景：

- 应用对自己的内部内容执行操作。例如：从攻击者处获取 URI 后，邮件应用会添加自己的内部 content provider 的数据（而不是外部照片）作为附件。
- 应用会充当代理，然后为攻击者访问其他应用的数据。例如：邮件应用会添加应用 X 的数据作为附件，而此类数据受到相关权限的保护，并且该权限通常禁止攻击者查看此类特定附件。这适用于会执行添加附件操作，但原本并不会将相应内容中继给攻击者的应用。

一种可能的攻击情景：创建一个恶意的图库或文件选择器，当存在漏洞的应用使用它时，它便会返回恶意 URI。

### 影响

利用此漏洞造成的影响因 ContentResolver 的相关使用情境而异。这可能会导致应用的受保护数据遭到未经授权方的窃取或篡改。

### 缓解措施

#### 一般措施

验证传入 URI。例如，使用包含预期机构的许可名单不失为一种好的做法。

#### URI 以存在漏洞的应用中不支持导出或受权限保护的 content provider 为目标

检查 URI 是否以您的应用为目标：

[Kotlin](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#kotlin)

```kotlin
fun belongsToCurrentApplication(ctx: Context, uri: Uri): Boolean {
    val authority: String = uri.authority.toString()
    val info: ProviderInfo =
        ctx.packageManager.resolveContentProvider(authority, 0)!!

    return ctx.packageName.equals(info.packageName)
}
```

[Java](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#java)
```java
boolean belongsToCurrentApplication(Context ctx, Uri uri){
    String authority = uri.getAuthority();
    ProviderInfo info = ctx.getPackageManager().resolveContentProvider(authority, 0);

    return ctx.getPackageName().equals(info.packageName);
}
```

或目标 provider 是否已导出：

[Kotlin](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#kotlin)
```kotlin
fun isExported(ctx: Context, uri: Uri): Boolean {
    val authority = uri.authority.toString()
    val info: ProviderInfo =
            ctx.packageManager.resolveContentProvider(authority, 0)!!

    return info.exported
}
```

[Java](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#java)
```java
boolean isExported(Context ctx, Uri uri){
    String authority = uri.getAuthority();
    ProviderInfo info = ctx.getPackageManager().resolveContentProvider(authority, 0);      

    return info.exported;
}
```
或是否已授予 URI 明确的权限（此检查基于以下假设：如果已授予访问数据的明确权限，相应 URI 便不存在恶意）：

[Kotlin](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#kotlin)
```kotlin
// grantFlag is one of: FLAG_GRANT_READ_URI_PERMISSION or FLAG_GRANT_WRITE_URI_PERMISSION
fun wasGrantedPermission(ctx: Context, uri: Uri?, grantFlag: Int): Boolean {
    val pid: Int = Process.myPid()
    val uid: Int = Process.myUid()
    return ctx.checkUriPermission(uri, pid, uid, grantFlag) ==
            PackageManager.PERMISSION_GRANTED
}
```

[Java](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#java)
```java
// grantFlag is one of: FLAG_GRANT_READ_URI_PERMISSION or FLAG_GRANT_WRITE_URI_PERMISSION
boolean wasGrantedPermission(Context ctx, Uri uri, int grantFlag){
    int pid = Process.myPid();
    int uid = Process.myUid();

    return ctx.checkUriPermission(uri, pid, uid, grantFlag) == PackageManager.PERMISSION_GRANTED;
}
```

#### URI 以信任有漏洞应用的其他应用中受权限保护的 ContentProvider 为目标。

此攻击与以下情况相关：

- 由应用定义并使用自定义权限或其他身份验证机制的应用生态系统。
- 权限代理攻击，即攻击者滥用存在漏洞且具有运行时权限（例如 READ_CONTACTS）的应用，以从 system provider 检索数据。

测试是否已授予 URI 权限：

[Kotlin](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#kotlin)

```kotlin
// grantFlag is one of: FLAG_GRANT_READ_URI_PERMISSION or FLAG_GRANT_WRITE_URI_PERMISSION
fun wasGrantedPermission(ctx: Context, uri: Uri?, grantFlag: Int): Boolean {
    val pid: Int = Process.myPid()
    val uid: Int = Process.myUid()
    return ctx.checkUriPermission(uri, pid, uid, grantFlag) ==
            PackageManager.PERMISSION_GRANTED
}
```

[Java](https://developer.android.com/privacy-and-security/risks/content-resolver?hl=zh-cn#java)

```java
// grantFlag is one of: FLAG_GRANT_READ_URI_PERMISSION or FLAG_GRANT_WRITE_URI_PERMISSION
boolean wasGrantedPermission(Context ctx, Uri uri, int grantFlag){
    int pid = Process.myPid();
    int uid = Process.myUid();

    return ctx.checkUriPermission(uri, pid, uid, grantFlag) == PackageManager.PERMISSION_GRANTED;
}
```

如果使用其他 content provider 时不需要授予权限（例如，当应用允许生态系统中的所有应用访问所有数据时），则明确禁止使用这些授权。

>**注意**：维护拒绝名单容易出错。拒绝软件包前缀可使此缓解措施更可靠。
