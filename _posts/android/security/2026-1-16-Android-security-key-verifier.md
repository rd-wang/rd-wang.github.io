---
title: Android-安全性-Android 系统密钥验证程序
date: 2026-1-16 11:16:20 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Key-Verifier
description: Android 密钥验证器为用户提供了一种统一且安全的方式，以验证他们在端到端加密 (E2EE) 应用中是否与正确的人进行通信。它通过可信且一致的系统用户界面，让用户确认联系人的公钥的真实性，从而保护用户免受中间人攻击。
math: true
---
Android 密钥验证器为用户提供了一种统一且安全的方式，以验证他们在端到端加密 (E2EE) 应用中是否与正确的人进行通信。它通过可信且一致的系统用户界面，让用户确认联系人的公钥的真实性，从而保护用户免受中间人攻击。

此功能由 [Key Verifier](https://play.google.com/store/apps/details?id=com.google.android.contactkeys&hl=zh-cn) 提供，密钥验证器是 Google 系统服务的一部分，并通过 Play 商店分发。它充当集中式的、设备端的端到端加密 (E2EE) 公钥存储库。

## 为什么要与 Key Verifier 集成

- **提供统一的用户体验**：与其构建自己的验证流程，不如启动系统的标准用户界面，从而为用户在所有应用程序上提供一致且值得信赖的体验。
- **提高用户信心**：清晰、系统支持的验证状态可确保用户对话的安全性和私密性。
- **减少开发开销**：将密钥验证界面、存储和状态管理的复杂性分流到系统服务。

> **重要提示**： 此服务用于验证密钥，不提供加密功能。 您的应用必须已实现 E2EE，并将密钥存储在用户设备上。

## 关键术语

- **lookupKey**：联系人的不透明持久标识符，存储在联系人提供程序的 [LOOKUP_KEY](https://developer.android.com/identity/providers/contacts-provider?hl=zh-cn#ContactBasics) 列中。与 `contact ID` 不同，即使基础联系人详细信息发生更改或合并，`lookupKey` 仍保持稳定，因此建议使用它来引用联系人。
- **accountId**：这是设备上用户帐户的应用程序专属标识符。此 ID 由您的应用程序定义，用于区分单个用户可能拥有的多个帐户。此信息会显示在用户界面中，建议使用有意义的信息，例如电话号码、电子邮件地址或用户名。
- **deviceId**：用户帐户关联的特定设备的唯一标识符。这允许用户拥有多个设备，每个设备都有自己的一组加密密钥。不一定代表物理设备，但可以用来区分同一账户使用的多个密钥。

## 使用入门

在开始之前，请设置您的应用以与密钥验证器服务通信。

**声明权限**：在 AndroidManifest.xml 中，声明以下权限。您还必须在运行时向用户请求这些权限。

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_CONTACTS" />
```

**获取客户端实例**：获取 `ContactKeys` 的实例，这是 API 的入口点。

```kotlin
import com.google.android.gms.contactkeys.ContactKeys

val contactKeyClient = ContactKeys.getClient(context)
```

## 面向即时通讯应用开发者的指南

作为即时通讯应用开发者，您的主要职责是将用户的公钥及其联系人的密钥发布到密钥验证服务。

### 发布用户的公钥

为了让其他人能够找到并验证您的用户身份，请将您的公钥发布到设备端密钥库。为了进一步提高安全性，建议您在 [Android Keystore](https://developer.android.com/privacy-and-security/keystore?hl=zh-cn) 中创建密钥。

```kotlin
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.tasks.Tasks

suspend fun publishSelfKey(
    contactKeyClient: ContactKeyClient,
    accountId: String,
    deviceId: String,
    publicKey: ByteArray
) {
    try {
        Tasks.await(
          contactKeyClient.updateOrInsertE2eeSelfKey(
            deviceId,
            accountId,
            publicKey
          )
        )
        // Self key published successfully.
    } catch (e: Exception) {
        // Handle error.
    }
}
```

### 将公钥与联系人相关联

当您的应用收到用户某位联系人的公钥时，您必须将其存储在中央存储库中，并将其与该联系人关联起来。这样可以验证密钥，并允许其他应用显示联系人的验证状态。为此，您需要从 Android 联系人提供程序获取联系人的 [lookupKey](https://developer.android.com/identity/providers/contacts-provider?hl=zh-cn#ContactBasics)。这通常会在从密钥分发服务器获取密钥或定期同步本地密钥时触发。

```kotlin
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.tasks.Tasks

suspend fun storeContactKey(
    contactKeyClient: ContactKeyClient,
    contactLookupKey: String,
    contactAccountId: String,
    contactDeviceId: String,
    contactPublicKey: ByteArray
) {
    try {
        Tasks.await(
            contactKeyClient.updateOrInsertE2eeContactKey(
                contactLookupKey,
                contactDeviceId,
                contactAccountId,
                contactPublicKey
            )
        )
        // Contact's key stored successfully.
    } catch (e: Exception) {
        // Handle error.
    }
}
```

> **注意**： 您可以使用接受查找键列表的 `updateOrInsertE2eeContactKey` 重载，将一个键与多个联系人关联起来，该重载接受查找键列表。

### 检索密钥和验证状态

发布密钥后，用户可以通过扫描实体二维码来验证密钥。应用的界面应反映对话是否使用经过验证的密钥。每个密钥都有一个验证状态，您可以使用该状态来更新界面。

**了解验证状态**：

- `UNVERIFIED`：这是每个新密钥的默认状态。这意味着密钥存在，但用户尚未确认其真实性。在界面中，您应将此视为中性状态，通常不显示任何特殊指示器。
    
- `VERIFIED`：此状态表示高度信任。这意味着用户已成功完成验证流程（例如扫描二维码），并确认密钥属于预期联系人。在用户界面中，您应该显示一个清晰、积极的指示器，例如绿色对勾或盾牌。
    
- `VERIFICATION_FAILED`：这是一个警告状态。这意味着与该联系人关联的密钥与之前验证过的密钥不匹配。这种情况可能发生在联系人获得新设备时，但也可能表明存在潜在的安全风险。在用户界面中，以醒目的警告提醒用户，并建议他们在发送敏感信息之前重新验证。
    

您可以检索与联系人关联的所有密钥的汇总状态。建议使用 `VerificationState.leastVerifiedFrom()`  来解析状态，当存在多个密钥时，它会正确地优先考虑 `VERIFICATION_FAILED` 而不是 `VERIFIED`。

- **获取联系人级别的汇总状态**

```kotlin
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.contactkeys.constants.VerificationState
import com.google.android.gms.tasks.Tasks

suspend fun displayContactVerificationStatus(
    contactKeyClient: ContactKeyClient,
    contactLookupKey: String
) {
    try {
        val keysResult = Tasks.await(contactKeyClient.getAllE2eeContactKeys(contactLookupKey))
        val states =
          keysResult.keys.map { VerificationState.fromState(it.localVerificationState) }
        val contactStatus = VerificationState.leastVerifiedFrom(states)
        updateUi(contactLookupKey, contactStatus)
    } catch (e: Exception) {
        // Handle error.
    }
}
```

- **获取账号级汇总状态**

```kotlin
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.contactkeys.constants.VerificationState
import com.google.android.gms.tasks.Tasks

suspend fun displayAccountVerificationStatus(
    contactKeyClient: ContactKeyClient,
    accountId: String
) {
    try {
        val keys = Tasks.await(contactKeyClient.getE2eeAccountKeysForAccount(accountId))
        val states = keys.map { VerificationState.fromState(it.localVerificationState) }
        val accountStatus = VerificationState.leastVerifiedFrom(states)
        updateUi(accountId, accountStatus)
    } catch (e: Exception) {
        // Handle error.
    }
}
```

### 实时观察key变化

为确保应用的界面始终显示正确的信任状态，您应监听更新。建议使用基于 Flow 的 API，每当订阅账号的密钥添加、移除或验证状态发生变化时，该 API 都会发出新的密钥列表。 这对于保持群组对话的成员列表处于最新状态尤为有用。以下情形可能会导致密钥的验证状态发生变化：

- 用户成功完成验证流程（例如，扫描二维码）。
- 联系人的密钥已修改，导致其不再与之前验证的值匹配。

```kotlin
fun observeKeyUpdates(contactKeyClient: ContactKeyClient, accountIds: List<String>) {
    lifecycleScope.launch {
        contactKeyClient.getAccountContactKeysFlow(accountIds)
            .collect { updatedKeys ->
                // A key was added, removed, or updated.
                // Refresh your app's UI and internal state.
                refreshUi(updatedKeys)
            }
    }
}
```

#### 进行现场密钥验证

用户验证密钥的最安全方式是面对面验证，通常通过扫描二维码或比较一系列数字来完成。Key Verifier 应用为此流程提供了标准界面流程，您的应用可以启动这些流程。在尝试验证后，API 会自动更新密钥的验证状态，如果您正在观察密钥更新，您的应用会收到通知。

- **启动用户所选联系人的密钥验证流程** 使用所选联系人的 `lookupKey` 启动由 `getScanQrCodeIntent` 提供的 `PendingIntent`。用户界面允许用户验证给定联系人的所有密钥。

```kotlin
import android.app.ActivityOptions
import android.app.PendingIntent
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.tasks.Tasks

suspend fun initiateVerification(contactKeyClient: ContactKeyClient, lookupKey: String) {
    try {
        val pendingIntent = Tasks.await(contactKeyClient.getScanQrCodeIntent(lookupKey))
        val options =
          ActivityOptions.makeBasic()
            .setPendingIntentBackgroundActivityStartMode(
              ActivityOptions.MODE_BACKGROUND_ACTIVITY_START_ALLOWED
            )
            .toBundle()
        pendingIntent.send(options)
    } catch (e: Exception) {
        // Handle error.
    }
}
```

- **启动用户所选帐户的密钥验证流程** 如果用户想要验证未直接与联系人关联的账号（或联系人的特定账号），您可以启动 `getScanQrCodeIntentForAccount` 提供的 `PendingIntent`。这通常用于您自己的应用程序的包名和 account ID。

```kotlin
import android.app.ActivityOptions
import android.app.PendingIntent
import com.google.android.gms.contactkeys.ContactKeyClient
import com.google.android.gms.tasks.Tasks

suspend fun initiateVerification(contactKeyClient: ContactKeyClient, packageName: String, accountId: String) {
    try {
        val pendingIntent = Tasks.await(contactKeyClient.getScanQrCodeIntentForAccount(packageName, accountId))
        // Allow activity start from background on Android SDK34+
        val options =
          ActivityOptions.makeBasic()
            .setPendingIntentBackgroundActivityStartMode(
              ActivityOptions.MODE_BACKGROUND_ACTIVITY_START_ALLOWED
            )
            .toBundle()
        pendingIntent.send(options)
    } catch (e: Exception) {
        // Handle error.
    }
}
```

