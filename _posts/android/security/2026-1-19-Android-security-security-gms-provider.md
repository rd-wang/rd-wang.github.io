---
title: Android-安全性-更新安全提供程序以防范 SSL 攻击
date: 2026-1-19 16:24:56 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Network
description:
math: true
---
Android 依赖安全 [`Provider`](https://developer.android.com/reference/java/security/Provider?hl=zh-cn) 提供安全的网络通信。但是，默认安全提供程序偶尔也会出现漏洞。为了防范这些漏洞， [Google Play 服务](https://developer.android.com/google/play-services?hl=zh-cn)提供了一种自动更新设备安全提供商的方法，以防范已知的漏洞利用。通过调用 Google Play 服务方法，您可以确保您的应用在具有最新更新的设备上运行，从而防止已知的漏洞利用。

例如，OpenSSL ([CVE-2014-0224](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0224)) 中发现了一个漏洞，该漏洞可能使应用程序容易受到路径攻击，攻击者可以在双方不知情的情况下解密安全流量。Google Play 服务 5.0 版本提供了一个修复程序，但应用必须检查此修复程序是否已安装。通过使用 Google Play 服务的方法，您可以确保您的应用运行在能够抵御此类攻击的设备上。

>**注意：** 更新设备的安全 `Provider` 不会更新 [`android.net.SSLCertificateSocketFactory`](https://developer.android.com/reference/android/net/SSLCertificateSocketFactory?hl=zh-cn)，因此它仍然存在漏洞。我们建议应用开发者使用高级方法与加密进行交互，而不是使用已废弃的类（例如 [`HttpsURLConnection`](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection?hl=zh-cn)）。

## 使用 ProviderInstaller 为安全提供程序打补丁

如需更新设备的安全提供程序，请使用 [`ProviderInstaller`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller?hl=zh-cn) 类。通过调用该类的 [`installIfNeeded()`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller?hl=zh-cn#installIfNeeded\(android.content.Context\))（或 [`installIfNeededAsync()`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller?hl=zh-cn#installIfNeededAsync\(android.content.Context,%20com.google.android.gms.security.ProviderInstaller.ProviderInstallListener\))）方法，您可以验证安全提供程序是否处于最新状态（如果需要，请对其进行更新）。本部分概略介绍了这些选项。有关更多详细步骤和示例，请参阅以下部分。

当您调用 `installIfNeeded()` 时，`ProviderInstaller` 将执行以下操作：

- 如果设备的 `Provider` 已成功更新（或者已经处于最新状态），此方法将返回而不会抛出异常。
- 如果设备的 Google Play 服务库已过期，此方法会抛出 [`GooglePlayServicesRepairableException`](https://developers.google.com/android/reference/com/google/android/gms/common/GooglePlayServicesRepairableException?hl=zh-cn)。然后，应用可以捕获此异常，并向用户显示相应的对话框以更新 Google Play 服务。
- 如果出现不可恢复的错误，此方法将抛出 [`GooglePlayServicesNotAvailableException`](https://developers.google.com/android/reference/com/google/android/gms/common/GooglePlayServicesNotAvailableException.html?hl=zh-cn) 以表明它无法更新 `Provider`。然后，应用可以捕获异常，并选择相应的操作，例如显示标准的[修复流程图](https://developers.google.com/android/reference/com/google/android/gms/common/SupportErrorDialogFragment.html?hl=zh-cn)。

`installIfNeededAsync()` 方法的行为方式与之相似，只是不会抛出异常，而是会调用相应的回调方法以表明更新成功还是失败。

如果安全提供程序已经处于最新状态，`installIfNeeded()` 需要的时间微不足道。如果此方法需要安装新的 `Provider`，所需的时间从 30-50 毫秒（在较新的设备上）到 350 毫秒（在较旧的设备上）不等。为避免影响用户体验，请执行以下操作：

- 在加载线程时立即从后台网络线程调用 `installIfNeeded()`，而不是等待线程尝试使用网络。（多次调用该方法没有任何坏处，因为如果安全提供程序不需要更新，它会立即返回。）
- 如果用户体验受线程拦截影响（例如，如果调用来自 UI 线程中的某个 activity），请调用此方法的异步版本，即 `installIfNeededAsync()`。（如果执行此操作，您需要等待操作完成后再尝试任何安全的通信。`ProviderInstaller` 会调用监听器的 [`onProviderInstalled()`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener.html?hl=zh-cn#onProviderInstalled\(\)) 方法以指示成功。）

> **警告：** 如果 `ProviderInstaller` 无法安装已更新的 `Provider`，您设备的安全提供程序可能会受到已知漏洞攻击。您的应用应表现为好像所有 HTTP 通信都未加密。

在更新 `Provider` 后，对安全 API（包括 SSL API）的所有调用均通过它进行路由。（但是，这不适用于 `android.net.SSLCertificateSocketFactory`，其仍然容易受到类似 [CVE-2014-0224](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0224) 等漏洞的攻击。）

### 同步打补丁

为安全提供程序打补丁的最简单方法是调用同步方法 `installIfNeeded()`。如果在等待操作完成时用户体验不会受到线程拦截影响，该方法会非常适用。

例如，下面展示了可以更新安全提供程序的 [worker](https://developer.android.com/topic/libraries/architecture/workmanager/basics?hl=zh-cn) 实现。由于 worker 在后台运行，因此，如果正在等待安全提供程序更新，线程就可以实施拦截。worker 会调用 `installIfNeeded()` 来更新安全提供程序。如果该方法正常返回，则 worker 知道安全提供程序处于最新状态。如果该方法引发异常，则 worker 可以进行适当的操作（例如提示用户更新 Google Play 服务）。

[Kotlin](https://developer.android.com/privacy-and-security/security-gms-provider?hl=zh-cn#kotlin)
```kotlin
/**
 * Sample patch Worker using {@link ProviderInstaller}.
 */
class PatchWorker(appContext: Context, workerParams: WorkerParameters): Worker(appContext, workerParams) {

  override fun doWork(): Result {
        try {
            ProviderInstaller.installIfNeeded(context)
        } catch (e: GooglePlayServicesRepairableException) {

            // Indicates that Google Play services is out of date, disabled, etc.

            // Prompt the user to install/update/enable Google Play services.
            GoogleApiAvailability.getInstance()
                    .showErrorNotification(context, e.connectionStatusCode)

            // Notify the WorkManager that a soft error occurred.
            return Result.failure()

        } catch (e: GooglePlayServicesNotAvailableException) {
            // Indicates a non-recoverable error; the ProviderInstaller can't
            // install an up-to-date Provider.

            // Notify the WorkManager that a hard error occurred.
            return Result.failure()
        }

        // If this is reached, you know that the provider was already up to date
        // or was successfully updated.
        return Result.success()
    }
}
```
[Java](https://developer.android.com/privacy-and-security/security-gms-provider?hl=zh-cn#java)

```java
/**
 * Sample patch Worker using {@link ProviderInstaller}.
 */
public class PatchWorker extends Worker {

  ...

  @Override
  public Result doWork() {
    try {
      ProviderInstaller.installIfNeeded(getContext());
    } catch (GooglePlayServicesRepairableException e) {

      // Indicates that Google Play services is out of date, disabled, etc.

      // Prompt the user to install/update/enable Google Play services.
      GoogleApiAvailability.getInstance()
              .showErrorNotification(context, e.connectionStatusCode)

      // Notify the WorkManager that a soft error occurred.
      return Result.failure();

    } catch (GooglePlayServicesNotAvailableException e) {
      // Indicates a non-recoverable error; the ProviderInstaller can't
      // install an up-to-date Provider.

      // Notify the WorkManager that a hard error occurred.
      return Result.failure();
    }

    // If this is reached, you know that the provider was already up to date
    // or was successfully updated.
    return Result.success();
  }
}
```

### 异步打补丁

更新安全提供程序最多需要 350 毫秒的时间（在较旧的设备上）。如果在直接影响用户体验的线程（例如界面线程）上进行更新，您一定不希望通过同步调用更新提供程序，因为这会导致应用或设备在相应操作完成前处于冻结状态。请改为使用异步方法 `installIfNeededAsync()`。该方法通过调用回调指示其成功还是失败。

例如，下面是可以在界面线程的某个 activity 中更新安全提供程序的一些代码。此 activity 会调用 `installIfNeededAsync()` 以更新提供程序，并将自身指定为接收成功或失败通知的监听器。如果安全提供程序为最新或已成功更新，将调用此 activity 的 [`onProviderInstalled()`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener.html?hl=zh-cn#onProviderInstalled\(\)) 方法，且 activity 知道通信是安全的。如果提供程序无法更新，将调用此 activity 的 [`onProviderInstallFailed()`](https://developers.google.com/android/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener.html?hl=zh-cn#onProviderInstallFailed\(int,%20android.content.Intent\)) 方法，且 activity 可以进行适当的操作（如提示用户更新 Google Play 服务）。

[Kotlin](https://developer.android.com/privacy-and-security/security-gms-provider?hl=zh-cn#kotlin)
```kotlin
private const val ERROR_DIALOG_REQUEST_CODE = 1

/**
 * Sample activity using {@link ProviderInstaller}.
 */
class MainActivity : Activity(), ProviderInstaller.ProviderInstallListener {

    private var retryProviderInstall: Boolean = false

    // Update the security provider when the activity is created.
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ProviderInstaller.installIfNeededAsync(this, this)
    }

    /**
     * This method is only called if the provider is successfully updated
     * (or is already up to date).
     */
    override fun onProviderInstalled() {
        // Provider is up to date; app can make secure network calls.
    }

    /**
     * This method is called if updating fails. The error code indicates
     * whether the error is recoverable.
     */
    override fun onProviderInstallFailed(errorCode: Int, recoveryIntent: Intent) {
        GoogleApiAvailability.getInstance().apply {
            if (isUserResolvableError(errorCode)) {
                // Recoverable error. Show a dialog prompting the user to
                // install/update/enable Google Play services.
                showErrorDialogFragment(this@MainActivity, errorCode, ERROR_DIALOG_REQUEST_CODE) {
                    // The user chose not to take the recovery action.
                    onProviderInstallerNotAvailable()
                }
            } else {
                onProviderInstallerNotAvailable()
            }
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int,
                                  data: Intent) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == ERROR_DIALOG_REQUEST_CODE) {
            // Adding a fragment via GoogleApiAvailability.showErrorDialogFragment
            // before the instance state is restored throws an error. So instead,
            // set a flag here, which causes the fragment to delay until
            // onPostResume.
            retryProviderInstall = true
        }
    }

    /**
     * On resume, check whether a flag indicates that the provider needs to be
     * reinstalled.
     */
    override fun onPostResume() {
        super.onPostResume()
        if (retryProviderInstall) {
            // It's safe to retry installation.
            ProviderInstaller.installIfNeededAsync(this, this)
        }
        retryProviderInstall = false
    }

    private fun onProviderInstallerNotAvailable() {
        // This is reached if the provider can't be updated for some reason.
        // App should consider all HTTP communication to be vulnerable and take
        // appropriate action.
    }
}
```
[Java](https://developer.android.com/privacy-and-security/security-gms-provider?hl=zh-cn#java)

```java
/**
 * Sample activity using {@link ProviderInstaller}.
 */
public class MainActivity extends Activity
    implements ProviderInstaller.ProviderInstallListener {

  private static final int ERROR_DIALOG_REQUEST_CODE = 1;

  private boolean retryProviderInstall;

  // Update the security provider when the activity is created.
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ProviderInstaller.installIfNeededAsync(this, this);
  }

  /**
   * This method is only called if the provider is successfully updated
   * (or is already up to date).
   */
  @Override
  protected void onProviderInstalled() {
    // Provider is up to date; app can make secure network calls.
  }

  /**
   * This method is called if updating fails. The error code indicates
   * whether the error is recoverable.
   */
  @Override
  protected void onProviderInstallFailed(int errorCode, Intent recoveryIntent) {
    GoogleApiAvailability availability = GoogleApiAvailability.getInstance();
    if (availability.isUserRecoverableError(errorCode)) {
      // Recoverable error. Show a dialog prompting the user to
      // install/update/enable Google Play services.
      availability.showErrorDialogFragment(
          this,
          errorCode,
          ERROR_DIALOG_REQUEST_CODE,
          new DialogInterface.OnCancelListener() {
            @Override
            public void onCancel(DialogInterface dialog) {
              // The user chose not to take the recovery action.
              onProviderInstallerNotAvailable();
            }
          });
    } else {
      // Google Play services isn't available.
      onProviderInstallerNotAvailable();
    }
  }

  @Override
  protected void onActivityResult(int requestCode, int resultCode,
      Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == ERROR_DIALOG_REQUEST_CODE) {
      // Adding a fragment via GoogleApiAvailability.showErrorDialogFragment
      // before the instance state is restored throws an error. So instead,
      // set a flag here, which causes the fragment to delay until
      // onPostResume.
      retryProviderInstall = true;
    }
  }

  /**
  * On resume, check whether a flag indicates that the provider needs to be
  * reinstalled.
  */
  @Override
  protected void onPostResume() {
    super.onPostResume();
    if (retryProviderInstall) {
      // It's safe to retry installation.
      ProviderInstaller.installIfNeededAsync(this, this);
    }
    retryProviderInstall = false;
  }

  private void onProviderInstallerNotAvailable() {
    // This is reached if the provider can't be updated for some reason.
    // App should consider all HTTP communication to be vulnerable and take
    // appropriate action.
  }
}
```

