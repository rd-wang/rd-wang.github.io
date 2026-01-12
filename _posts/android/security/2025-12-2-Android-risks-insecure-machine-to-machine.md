---
title: Android-安全性-安全风险-不安全的机器间通信设置
date: 2025-12-2 08:19:07 +0800
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
description: 机器对机器通信信道的错误使用或配置可能会使用户设备暴露于不受信任的通信尝试之中。
math: true
---

# 不安全的机器间通信设置

**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

现在，很多应用程序都实现了允许用户使用射频 (RF) 通信或有线连接来传输数据或与其他设备交互的功能。Android 中用于此目的最常用的技术是经典蓝牙（蓝牙 BR/EDR）、低功耗蓝牙（BLE）、Wifi P2P、NFC 和 USB。

这些技术通常应用于需要与智能家居配件、健康监测设备、公共交通信息亭、支付终端和其他安卓设备进行通信的应用程序中。

与其他任何渠道一样，机器对机器通信也容易受到旨在破坏两个或多个设备之间建立的信任边界的攻击。恶意用户可以利用设备冒充等技术，对通信信道发起多种攻击。

Android 为开发者提供了用于配置机器间通信的特定[API](https://developer.android.com/develop/connectivity/overview?hl=zh-cn) 。

使用这些API时应格外谨慎，因为在实现通信协议过程中出现的错误可能会导致用户或设备数据暴露给未经授权的第三方。在最糟糕的情况下，攻击者可能能够远程控制一台或多台设备，从而完全访问设备上的内容。

## 影响

影响程度可能因应用程序中采用的设备间通信技术而异。

机器对机器通信信道的错误使用或配置可能会使用户设备暴露于不受信任的通信尝试之中。这可能导致设备容易受到中间人攻击 (MiTM)、命令注入、拒绝服务攻击 (DoS) 或冒充攻击等其他攻击。

## 风险：通过无线通道窃听敏感数据

在实现机器间通信机制时，应仔细考虑所使用的技术和待传输的数据类型。虽然有线连接在实践中更安全，因为它需要在相关设备之间建立物理链路，但使用射频的通信协议（例如经典蓝牙、BLE、NFC 和 Wi-Fi P2P）仍可能被拦截。攻击者可以冒充参与数据交换的终端或接入点，拦截无线通信，从而获取敏感的用户数据。此外，如果设备上安装的恶意应用程序获得了通信相关的[运行时权限](https://developer.android.com/guide/topics/permissions/overview#runtime)，则可以通过读取系统消息缓冲区来获取设备间交换的数据。

### 缓解措施

如果应用程序确实需要通过无线信道进行机器对机器的敏感数据交换，那么应该在应用程序代码中实现应用层安全解决方案，例如加密。这将防止攻击者嗅探通信信道并以明文形式获取交换的数据。更多资源，请参阅[加密](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn)文档。

---

## 风险：无线恶意数据注入

无线机器对机器通信信道（经典蓝牙、BLE、NFC、Wifi P2P）可能会被恶意数据篡改。技术足够娴熟的攻击者可以识别正在使用的通信协议，并篡改数据交换流程，例如冒充其中一个端点，发送专门构造的有效载荷。这种恶意流量可能会降低应用程序的功能，在最坏的情况下，会导致应用程序和设备出现意外行为，或者导致拒绝服务攻击、命令注入或设备接管等攻击。

### 缓解措施

Android 为开发者提供了[强大的 API](https://developer.android.com/develop/connectivity/overview?hl=zh-cn) 机器对机器通信，如传统蓝牙、BLE、NFC 和 Wifi P2P。这些措施应与精心实施的数据验证逻辑相结合，以清理两个设备之间交换的任何数据。

该解决方案应在应用程序级别实施，并应包含检查，以验证数据是否具有预期的长度、格式，以及是否包含应用程序可以解释的有效有效负载。

以下代码片段展示了示例数据验证逻辑。该逻辑基于 Android 开发者提供的[蓝牙](https://developer.android.com/develop/connectivity/bluetooth/transfer-data#example)数据传输示例实现：

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#kotlin)
```kotlin
class MyThread(private val mmInStream: InputStream, private val handler: Handler) : Thread() {

    private val mmBuffer = ByteArray(1024)
      override fun run() {
        while (true) {
            try {
                val numBytes = mmInStream.read(mmBuffer)
                if (numBytes > 0) {
                    val data = mmBuffer.copyOf(numBytes)
                    if (isValidBinaryData(data)) {
                        val readMsg = handler.obtainMessage(
                            MessageConstants.MESSAGE_READ, numBytes, -1, data
                        )
                        readMsg.sendToTarget()
                    } else {
                        Log.w(TAG, "Invalid data received: $data")
                    }
                }
            } catch (e: IOException) {
                Log.d(TAG, "Input stream was disconnected", e)
                break
            }
        }
    }

    private fun isValidBinaryData(data: ByteArray): Boolean {
        if (// Implement data validation rules here) {
            return false
        } else {
            // Data is in the expected format
            return true
        }
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#java)

```java
public void run() {
            mmBuffer = new byte[1024];
            int numBytes; // bytes returned from read()
            // Keep listening to the InputStream until an exception occurs.
            while (true) {
                try {
                    // Read from the InputStream.
                    numBytes = mmInStream.read(mmBuffer);
                    if (numBytes > 0) {
                        // Handle raw data directly
                        byte[] data = Arrays.copyOf(mmBuffer, numBytes);
                        // Validate the data before sending it to the UI activity
                        if (isValidBinaryData(data)) {
                            // Data is valid, send it to the UI activity
                            Message readMsg = handler.obtainMessage(
                                    MessageConstants.MESSAGE_READ, numBytes, -1,
                                    data);
                            readMsg.sendToTarget();
                        } else {
                            // Data is invalid
                            Log.w(TAG, "Invalid data received: " + data);
                        }
                    }
                } catch (IOException e) {
                    Log.d(TAG, "Input stream was disconnected", e);
                    break;
                }
            }
        }

        private boolean isValidBinaryData(byte[] data) {
            if (// Implement data validation rules here) {
                return false;
            } else {
                // Data is in the expected format
                return true;
           }
    }
```

---

## 风险：USB 恶意数据注入

恶意用户可能会利用两台设备之间的 USB 连接来拦截通信。在这种情况下，所需的物理链路构成了一层额外的安全保障，因为攻击者需要接触到连接终端的线缆才能窃听任何消息。另一种攻击途径是将不受信任的 USB 设备（无论是有意还是无意地）插入设备。

如果应用程序使用 PID/VID 来过滤 USB 设备以触发特定的应用内功能，攻击者可能通过冒充合法设备来篡改通过 USB 通道传输的数据。此类攻击可能允许恶意用户向设备发送按键指令或执行应用程序活动，在最坏的情况下，可能导致远程代码执行或下载恶意软件。

### 缓解措施

应实现应用层验证逻辑。该逻辑应过滤通过 USB 发送的数据，检查其长度、格式和内容是否符合应用场景。例如，心率监测器不应能够发送按键命令。

此外，如果条件允许，应考虑限制应用程序可以从USB设备接收的USB数据包数量。这可以防止恶意设备发起诸如橡皮鸭攻击之类的攻击。

可以通过创建一个新线程来检查缓冲区内容，例如，在执行以下操作时[`bulkTransfer`](https://developer.android.com/reference/android/hardware/usb/UsbDeviceConnection#bulkTransfer\(android.hardware.usb.UsbEndpoint,%20byte%5B%5D,%20int,%20int\))：

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#kotlin)
```kotlin
fun performBulkTransfer() {
    // Stores data received from a device to the host in a buffer
    val bytesTransferred = connection.bulkTransfer(endpointIn, buffer, buffer.size, 5000)

    if (bytesTransferred > 0) {
        if (//Checks against buffer content) {
            processValidData(buffer)
        } else {
            handleInvalidData()
        }
    } else {
        handleTransferError()
    }
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#java)

```java
public void performBulkTransfer() {
        //Stores data received from a device to the host in a buffer
        int bytesTransferred = connection.bulkTransfer(endpointIn, buffer, buffer.length, 5000);
        if (bytesTransferred > 0) {
            if (//Checks against buffer content) {
                processValidData(buffer);
            } else {
                handleInvalidData();
            }
        } else {
            handleTransferError();
        }
    }
```

---

## 特定风险

本节收集了需要非标准缓解策略或在某些 SDK 级别中已缓解的风险，此处仅供参考。

### 风险：蓝牙 - 可检测性时间不正确

正如[Android 开发者蓝牙文档](https://developer.android.com/develop/connectivity/bluetooth/find-bluetooth-devices#enable-discoverability)中所述，在应用程序中配置蓝牙接口时，使用 [`startActivityForResult(Intent, int)`](https://developer.android.com/reference/android/app/Activity#startActivityForResult\(android.content.Intent,%20int\))启用设备可发现性的方法并将设置值[`EXTRA_DISCOVERABLE_DURATION`](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter#EXTRA_DISCOVERABLE_DURATION)设为零，将使设备在应用程序于后台或前台运行时始终处于可发现状态。根据[经典的蓝牙规范](https://www.bluetooth.com/learn-about-bluetooth/tech-overview/)，可发现的设备会持续广播特定的发现消息，允许其他设备检索设备数据或与之连接。在这种情况下，恶意第三方可以拦截这些消息并连接到 Android 设备。一旦连接成功，攻击者就可以执行进一步的攻击，例如数据窃取、拒绝服务攻击 (DoS) 或命令注入。

#### 缓解措施

该参数`EXTRA_DISCOVERABLE_DURATION`不应设置为零。如果 `EXTRA_DISCOVERABLE_DURATION`未设置此参数，Android 默认会将设备设置为可发现状态 2 分钟。该 `EXTRA_DISCOVERABLE_DURATION`参数的最大可设置值为 2 小时（7200 秒）。建议根据应用场景，将可发现状态的持续时间设置为尽可能短。

---

### 风险：NFC - 克隆的 intent 过滤器

恶意应用程序可以注册 intent 过滤器来读取特定的 NFC 标签或支持 NFC 的设备。这些过滤器可以复制合法应用程序定义的过滤器，使攻击者能够读取交换的 NFC 数据的内容。需要注意的是，当两个活动为特定的 NFC 标签指定相同的意图过滤器时，会显示[活动选择器](https://developer.android.com/develop/connectivity/nfc/nfc#dispatching)，因此用户仍然需要选择恶意应用程序才能使攻击成功。尽管如此，将意图过滤器与伪装技术结合使用，这种情况仍然可能发生。这种攻击仅在通过 NFC 交换的数据高度敏感的情况下才具有重要意义。

#### 缓解措施

[在应用程序中实现 NFC 读取功能时，可以将意图过滤器与Android 应用程序记录](https://developer.android.com/develop/connectivity/nfc/nfc#aar)(AAR)结合使用。将 AAR 记录嵌入 NDEF 消息中，可以有效确保只有合法的应用程序及其关联的 NDEF 处理活动才会启动。这将防止未经授权的应用程序或活动读取通过 NFC 交换的高度敏感的标签或设备数据。

---

### 风险：NFC - 缺少 NDEF 消息验证

当 Android 设备接收来自 [NFC 标签](https://developer.android.com/develop/connectivity/nfc/nfc?hl=zh-cn#tag-dispatch)或支持NFC功能的设备的数据时，系统会自动触发已配置为处理该NDEF消息的应用程序或特定 activity。 根据应用程序中实现的逻辑，标签中包含的数据或从设备接收到的数据可以传递给其他 activity，以触发进一步的操作，例如打开网页。

如果应用缺少 NDEF 消息内容验证，攻击者可能会使用支持 NFC 的设备或 NFC 标签向应用程序注入恶意有效载荷，从而导致意外行为，例如恶意文件下载、命令注入或 DoS 攻击。

#### 缓解措施

在将收到的 NDEF 消息发送给任何其他应用组件之前，应验证消息中的数据是否符合预期格式并包含预期信息。这可以避免恶意数据未经过滤地传递给其他应用程序组件，从而降低因篡改 NFC 数据而导致意外行为或攻击的风险。

以下代码片段展示了一个示例数据验证逻辑，该逻辑以方法的形式实现，参数为 NDEF 消息及其在消息数组中的索引。此示例基于 Android 开发者的[示例](https://developer.android.com/develop/connectivity/nfc/nfc#obtain-info)实现，用于从扫描的 NFC NDEF 标签获取数据：

[Kotlin](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#kotlin)
```kotlin
//The method takes as input an element from the received NDEF messages array
fun isValidNDEFMessage(messages: Array<NdefMessage>, index: Int): Boolean {
    // Checks if the index is out of bounds
    if (index < 0 || index >= messages.size) {
        return false
    }
    val ndefMessage = messages[index]
    // Retrieves the record from the NDEF message
    for (record in ndefMessage.records) {
        // Checks if the TNF is TNF_ABSOLUTE_URI (0x03), if the Length Type is 1
        if (record.tnf == NdefRecord.TNF_ABSOLUTE_URI && record.type.size == 1) {
            // Loads payload in a byte array
            val payload = record.payload

            // Declares the Magic Number that should be matched inside the payload
            val gifMagicNumber = byteArrayOf(0x47, 0x49, 0x46, 0x38, 0x39, 0x61) // GIF89a

            // Checks the Payload for the Magic Number
            for (i in gifMagicNumber.indices) {
                if (payload[i] != gifMagicNumber[i]) {
                    return false
                }
            }
            // Checks that the Payload length is, at least, the length of the Magic Number + The Descriptor
            if (payload.size == 13) {
                return true
            }
        }
    }
    return false
}
```
[Java](https://developer.android.com/privacy-and-security/risks/insecure-machine-to-machine?hl=zh-cn#java)

```java
//The method takes as input an element from the received NDEF messages array
    public boolean isValidNDEFMessage(NdefMessage[] messages, int index) {
        //Checks if the index is out of bounds
        if (index < 0 || index >= messages.length) {
            return false;
        }
        NdefMessage ndefMessage = messages[index];
        //Retrieve the record from the NDEF message
        for (NdefRecord record : ndefMessage.getRecords()) {
            //Check if the TNF is TNF_ABSOLUTE_URI (0x03), if the Length Type is 1
            if ((record.getTnf() == NdefRecord.TNF_ABSOLUTE_URI) && (record.getType().length == 1)) {
                //Loads payload in a byte array
                byte[] payload = record.getPayload();
                //Declares the Magic Number that should be matched inside the payload
                byte[] gifMagicNumber = {0x47, 0x49, 0x46, 0x38, 0x39, 0x61}; // GIF89a
                //Checks the Payload for the Magic Number
                for (int i = 0; i < gifMagicNumber.length; i++) {
                    if (payload[i] != gifMagicNumber[i]) {
                        return false;
                    }
                }
                //Checks that the Payload length is, at least, the length of the Magic Number + The Descriptor
                if (payload.length == 13) {
                    return true;
                }
            }
        }
        return false;
    }
```

---

## 资源

- [运行时权限](https://developer.android.com/guide/topics/permissions/overview?hl=zh-cn#runtime)
- [连接性指南](https://developer.android.com/develop/connectivity/overview?hl=zh-cn)
- [示例](https://developer.android.com/develop/connectivity/bluetooth/transfer-data?hl=zh-cn#example)
- [bulkTransfer](https://developer.android.com/reference/android/hardware/usb/UsbDeviceConnection?hl=zh-cn#bulkTransfer\(android.hardware.usb.UsbEndpoint,%20byte%5B%5D,%20int,%20int\))
- [加密](https://developer.android.com/privacy-and-security/cryptography?hl=zh-cn)
- [设置蓝牙](https://developer.android.com/develop/connectivity/bluetooth/setup?hl=zh-cn)
- [NFC 基础](https://developer.android.com/develop/connectivity/nfc/nfc?hl=zh-cn)
- [Android 应用记录](https://developer.android.com/develop/connectivity/nfc/nfc?hl=zh-cn#aar)
- [传统蓝牙规范](https://www.bluetooth.com/learn-about-bluetooth/tech-overview)
