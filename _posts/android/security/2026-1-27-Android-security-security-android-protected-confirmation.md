---
title: Android-安全性-Android Protected Confirmation
date: 2026-1-20 08:48:01 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
description: 为了帮助您确认用户发起敏感交易（例如付款）时的意图，搭载 Android 9（API 级别 28）或更高版本的受支持设备允许您使用 Android Protected Confirmation 功能。使用此工作流时，应用会向用户显示提示，请他们批准一个简短的声明，以再次确认他们完成敏感交易的意图。
math: true
---
为了帮助您确认用户发起敏感交易（例如付款）时的意图，搭载 Android 9（API 级别 28）或更高版本的受支持设备允许您使用 Android Protected Confirmation 功能。使用此工作流时，应用会向用户显示提示，请他们批准一个简短的声明，以再次确认他们完成敏感交易的意图。

如果用户接受该声明，应用就可以使用 Android 密钥库中的密钥对对话框中显示的消息进行签名。该签名具有很高的可信度，表示用户已看过声明并同意其内容。

> **注意：** Android 受保护的确认并非为用户提供安全信息通道。您的应用不能提供除 Android 平台本身提供的保密保障之外的任何保密保障。尤其需要注意的是，请勿使用此工作流程显示您通常不会在用户设备上显示的敏感信息。
> 
> 用户确认消息后，消息的完整性即可得到保证，但您的应用仍需使用传输中数据加密来保护签名消息的机密性。

要为您的应用提供高可靠性用户确认支持，请完成以下步骤：

1. 使用 [`KeyGenParameterSpec.Builder`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn) 类[生成非对称签名密钥](https://developer.android.com/training/articles/keystore?hl=zh-cn#GeneratingANewPrivateKey)。创建该密钥时，将 `true` 传入 [`setUserConfirmationRequired()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn#setUserConfirmationRequired\(boolean\)) 中。此外，调用 [`setAttestationChallenge()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn#setAttestationChallenge\(byte%5B%5D\)) 并传入由依赖方提供的合适私钥保护值。
    
2. 将新生成的密钥和密钥的证明证书注册到相应的信赖方。
    
3. 将交易详情发送到您的服务器，并让服务器生成并返回一个包含额外数据的二进制大对象 (BLOB)。额外数据可能包括待确认的数据或解析提示，例如提示字符串的区域设置。
    
    为了提高实现的安全性，该 BLOB 必须包含加密随机数以防范[重放攻击](https://www.pcmag.com/encyclopedia/term/50439/replay-attack)并消除交易歧义。
    
    >**注意：** 如果附加数据字段包含待确认数据，则依赖方必须验证与提示字符串一起发送的相同数据。Android 受保护的确认功能不会渲染额外数据，因此您的应用无法假定用户已确认这些数据。
    
1. 设置 [`ConfirmationCallback`](https://developer.android.com/reference/android/security/ConfirmationCallback?hl=zh-cn) 对象，让它在用户已接受确认对话框中显示的提示时通知应用：
			
	[Kotlin](https://developer.android.com/privacy-and-security/security-android-protected-confirmation?hl=zh-cn#kotlin)
	
	```kotlin
	class MyConfirmationCallback : ConfirmationCallback() {

	  override fun onConfirmed(dataThatWasConfirmed: ByteArray?) {
		  super.onConfirmed(dataThatWasConfirmed)
		  // Sign dataThatWasConfirmed using your generated signing key.
		  // By completing this process, you generate a signed statement.
	  }

	  override fun onDismissed() {
		  super.onDismissed()
		  // Handle case where user declined the prompt in the
		  // confirmation dialog.
	  }

	  override fun onCanceled() {
		  super.onCanceled()
		  // Handle case where your app closed the dialog before the user
		  // responded to the prompt.
	  }

	  override fun onError(e: Exception?) {
		  super.onError(e)
		  // Handle the exception that the callback captured.
	  }
  }
	```
	    
	[Java](https://developer.android.com/privacy-and-security/security-android-protected-confirmation?hl=zh-cn#java)
	
	```java
	public class MyConfirmationCallback extends ConfirmationCallback {
	
	  @Override
	  public void onConfirmed(@NonNull byte[] dataThatWasConfirmed) {
		  super.onConfirmed(dataThatWasConfirmed);
		  // Sign dataThatWasConfirmed using your generated signing key.
		  // By completing this process, you generate a signed statement.
	  }
	
	  @Override
	  public void onDismissed() {
		  super.onDismissed();
		  // Handle case where user declined the prompt in the
		  // confirmation dialog.
	  }
	
	  @Override
	  public void onCanceled() {
		  super.onCanceled();
		  // Handle case where your app closed the dialog before the user
		  // responded to the prompt.
	  }
	
	  @Override
	  public void onError(Throwable e) {
		  super.onError(e);
		  // Handle the exception that the callback captured.
	  }
	}
	```
	
	
	如果用户批准该对话框，则调用 `onConfirmed()` 回调。`dataThatWasConfirmed` BLOB 是一个 [CBOR 数据结构](http://cbor.io/)，其中包含用户看到的提示文本以及您传入 [`ConfirmationPrompt`](https://developer.android.com/reference/android/security/ConfirmationPrompt?hl=zh-cn) 构建器的额外数据，还包含其他详细信息。使用之前创建的密钥签署 `dataThatWasConfirmed` BLOB，然后将 BLOB 连同签名和交易详情回传给依赖方。
	
	**注意：** 由于密钥是使用 [`setUserConfirmationRequired()`](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder?hl=zh-cn#setUserConfirmationRequired\(boolean\)) 创建的，因此只能用于签署 `dataThatWasConfirmed` 参数中返回的数据。尝试签署任何其他类型的数据都不会成功。
	    
   为了充分利用 Android 受保护的确认提供的安全保障，依赖方必须在收到已签署的消息后执行以下步骤：
	    
	1. 检查消息上的签名以及签名密钥的认证证书链。
	2. 检查认证证书是否设置了 `TRUSTED_CONFIRMATION_REQUIRED` 标记，这表示签名密钥需要受信任的用户确认。如果签名密钥是 RSA 密钥，请确认它不具有 [`PURPOSE_ENCRYPT`](https://developer.android.com/reference/android/security/keystore/KeyProperties?hl=zh-cn#PURPOSE_ENCRYPT) 或 [`PURPOSE_DECRYPT`](https://developer.android.com/reference/android/security/keystore/KeyProperties?hl=zh-cn#PURPOSE_DECRYPT) 属性。
	3. 检查 `extraData` 以确保此确认消息属于新请求但尚未处理。此步骤可防范重放攻击。
	4. 解析 `promptText` 以获取有关已确认操作或请求的信息。请谨记，`promptText` 是用户实际确认的消息的唯一部分。依赖方绝不能假设 `extraData` 包含的待确认数据对应于 `promptText`。
    
5. 添加与以下代码段所示内容类似的逻辑，以显示对话框本身：
    
	[Kotlin](https://developer.android.com/privacy-and-security/security-android-protected-confirmation?hl=zh-cn#kotlin)
	
	```kotlin
	// This data structure varies by app type. This is an example.
	  data class ConfirmationPromptData(val sender: String,
			  val receiver: String, val amount: String)
	
	  val myExtraData: ByteArray = byteArrayOf()
	  val myDialogData = ConfirmationPromptData("Ashlyn", "Jordan", "$500")
	  val threadReceivingCallback = Executor { runnable -> runnable.run() }
	  val callback = MyConfirmationCallback()
	
	  val dialog = ConfirmationPrompt.Builder(context)
			  .setPromptText("${myDialogData.sender}, send
							  ${myDialogData.amount} to
							  ${myDialogData.receiver}?")
			  .setExtraData(myExtraData)
			  .build()
	  dialog.presentPrompt(threadReceivingCallback, callback)
	```
	    
	[Java](https://developer.android.com/privacy-and-security/security-android-protected-confirmation?hl=zh-cn#java)
	
	```java
	  // This data structure varies by app type. This is an example.
	  class ConfirmationPromptData {
		  String sender, receiver, amount;
		  ConfirmationPromptData(String sender, String receiver, String amount) {
			  this.sender = sender;
			  this.receiver = receiver;
			  this.amount = amount;
		  }
	  };
	  final int MY_EXTRA_DATA_LENGTH = 100;
	  byte[] myExtraData = new byte[MY_EXTRA_DATA_LENGTH];
	  ConfirmationPromptData myDialogData = new ConfirmationPromptData("Ashlyn", "Jordan", "$500");
	  Executor threadReceivingCallback = Runnable::run;
	  MyConfirmationCallback callback = new MyConfirmationCallback();
	  ConfirmationPrompt dialog = (new ConfirmationPrompt.Builder(getApplicationContext()))
			  .setPromptText("${myDialogData.sender}, send ${myDialogData.amount} to ${myDialogData.receiver}?")
			  .setExtraData(myExtraData)
			  .build();
	  dialog.presentPrompt(threadReceivingCallback, callback);
	```
	    
> 	**注意：** 包含全屏对话框的确认提示界面无法自定义，但框架会为您处理按钮文本的本地化。
    

## 其他资源

要详细了解 Android 受保护的确认，请参阅以下资源。

### 博客

- [Android 受保护的确认：进一步提高交易安全性](https://android-developers.googleblog.com/2018/10/android-protected-confirmation.html)