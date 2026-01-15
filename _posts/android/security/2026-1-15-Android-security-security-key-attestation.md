---
title: Android-安全性-验证由硬件支持的密钥对
date: 2026-1-15 14:27:19 +0800
categories:
  - Android
  - Security
tags:
  - Android
  - Security
  - Hardware-backed
  - Key-Pairs
description: 利用密钥认证，对于将应用中使用的密钥存储在设备硬件支持的密钥库中这一做法，您可以更加放心。分为几部分介绍了如何验证硬件支持的密钥的属性，以及如何解释认证证书的扩展数据。
math: true
---

>**注意**：在生产级环境中为设备验证硬件支持的密钥的属性前，您应确保设备支持硬件级密钥认证。为此，您需要确认认证证书链包含用 Google 认证根密钥签名的根证书，且[密钥描述](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#key_attestation_ext_schema_keydescription)数据结构中的 `attestationSecurityLevel` 元素被设置为 `TrustedEnvironment` 安全等级。
>此外，请务必验证证书链中的签名，并查看[证书吊销状态列表](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#certificate_status)，确认证书链中没有密钥被吊销。除非所有密钥均有效且根是 Google 根密钥，否则请勿完全信任认证。但是请注意，包含已吊销的证书的设备至少和仅支持软件认证的设备一样可信。在可信度方面，具有完全有效的认证是强有力的正面指标，但没有完全有效的认证则属于一般情况，并非是负面指标。

## 检索和验证硬件支持的密钥对 

在密钥认证期间，可以指定密钥对的别名并检索其证书链，证书链可用于验证该密钥对的属性。

如果设备支持硬件级密钥认证，将使用认证根密钥为此证书链中的根证书签名，而该根密钥已安全配置到设备的硬件支持的密钥库。

>**注意：** 在附带硬件级密钥认证、搭载 Android 7.0（API 级别 24）或更高版本以及预装 Google Play 服务的设备上，根证书使用 Google 认证根密钥签名。验证此根证书是否属于[根证书](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#root_certificate)部分中列出的证书之一。

如需实现密钥认证，请完成以下步骤：

1. 使用 [KeyStore](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn) 对象的 [getCertificateChain()](https://developer.android.com/reference/java/security/KeyStore?hl=zh-cn#getCertificateChain\(java.lang.String\)) 方法获取指向 X.509 证书链的引用，该证书链与硬件支持的密钥库关联。
2. 将证书发送到您信任的独立服务器进行验证。
    
    >**注意：** 请不要在密钥库所在的设备上完成以下验证流程。原因在于一旦相应设备上的 Android 系统遭到入侵，可能会导致验证流程信任不可信的内容。
    
3. 获取对您的工具集来说最适合的 X.509 证书链解析和验证库的引用。验证根公共证书是否可信，以及每个证书是否签署了证书链中的下一个证书。
    
4. 检查每个证书的[吊销状态](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#certificate_status)，确保所有证书都没有被吊销。
    
5. 可选择检查配置信息证书扩展（仅存在于较新的证书链中）。
    
    获取指向最适合您工具集的 CBOR 解析器库的引用。找到包含[配置信息证书扩展](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#provisioning_attestation_ext_schema)并且最接近根证书的证书。使用解析器从该证书中提取配置信息证书扩展数据。
    
    如需了解详情，请参阅[配置信息扩展数据架构](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#provisioning_attestation_ext_schema)部分。
    
6. 获取指向最适合您工具集的 ASN.1 解析器库的引用。找到包含[密钥认证证书扩展](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#key_attestation_ext_schema)并且最接近根证书的证书。如果存在配置信息证书扩展，则密钥认证证书扩展一定位于紧随其后的证书中。使用解析器从该证书中提取密钥认证证书扩展数据。
    
    >**注意：** 请勿假定密钥认证证书扩展位于证书链的叶证书中。扩展只有在证书链中首次出现时才可信。扩展的任何后续实例都不是由安全硬件发出的，而可能是对证书链进行扩展的攻击者在尝试为不受信任的密钥创建虚假认证时发出的。
    
    该[密钥认证示例](https://github.com/googlesamples/android-key-attestation)使用 [Bouncy Castle](https://www.bouncycastle.org/) 中的 ASN.1 解析器提取认证证书的扩展数据。在创建您自己的解析器时，您可以将此示例用作参考。
    
    如需了解详情，请参阅[密钥认证扩展数据架构](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#key_attestation_ext_schema)部分。
    
1. 请检查您在前面步骤中检索到的扩展数据，确保一致性，并与您希望在硬件支持的密钥中包含的一组值进行比较。

## 根证书 

认证的可信度取决于证书链的根证书。对于已通过预装 Google 一系列应用（包括 Google Play）所需测试的 Android 设备，以及搭载 Android 7.0（API 级别 24）或更高版本的设备，应使用 Google 硬件认证根证书签名的认证密钥。请注意，如果设备搭载低于 Android 8.0（API 级别 26）的系统，则不需要进行认证。根公钥如下所示：

```
  -----BEGIN PUBLIC KEY-----
  MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xU
  FmOr75gvMsd/dTEDDJdSSxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5j
  lRfdnJLmN0pTy/4lj4/7tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y
  //0rb+T+W8a9nsNL/ggjnar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73X
  pXyTqRxB/M0n1n/W9nGqC4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYI
  mQQcHtGl/m00QLVWutHQoVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB
  +TxywElgS70vE0XmLD+OJtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7q
  uvmag8jfPioyKvxnK/EgsTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgp
  Zrt3i5MIlCaY504LzSRiigHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7
  gLiMm0jhO2B6tUXHI/+MRPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82
  ixPvZtXQpUpuL12ab+9EaDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+
  NpUFgNPN9PvQi8WEg5UmAGMCAwEAAQ==
  -----END PUBLIC KEY-----
  ```

之前颁发的根证书
```
    -----BEGIN CERTIFICATE-----
    MIIFYDCCA0igAwIBAgIJAOj6GWMU0voYMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
    BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMTYwNTI2MTYyODUyWhcNMjYwNTI0MTYy
    ODUyWjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
    AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
    Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
    tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
    nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
    C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
    oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
    JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
    sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
    igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
    RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
    aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
    AGMCAwEAAaOBpjCBozAdBgNVHQ4EFgQUNmHhAHyIBQlRi0RsR/8aTMnqTxIwHwYD
    VR0jBBgwFoAUNmHhAHyIBQlRi0RsR/8aTMnqTxIwDwYDVR0TAQH/BAUwAwEB/zAO
    BgNVHQ8BAf8EBAMCAYYwQAYDVR0fBDkwNzA1oDOgMYYvaHR0cHM6Ly9hbmRyb2lk
    Lmdvb2dsZWFwaXMuY29tL2F0dGVzdGF0aW9uL2NybC8wDQYJKoZIhvcNAQELBQAD
    ggIBACDIw41L3KlXG0aMiS//cqrG+EShHUGo8HNsw30W1kJtjn6UBwRM6jnmiwfB
    Pb8VA91chb2vssAtX2zbTvqBJ9+LBPGCdw/E53Rbf86qhxKaiAHOjpvAy5Y3m00m
    qC0w/Zwvju1twb4vhLaJ5NkUJYsUS7rmJKHHBnETLi8GFqiEsqTWpG/6ibYCv7rY
    DBJDcR9W62BW9jfIoBQcxUCUJouMPH25lLNcDc1ssqvC2v7iUgI9LeoM1sNovqPm
    QUiG9rHli1vXxzCyaMTjwftkJLkf6724DFhuKug2jITV0QkXvaJWF4nUaHOTNA4u
    JU9WDvZLI1j83A+/xnAJUucIv/zGJ1AMH2boHqF8CY16LpsYgBt6tKxxWH00XcyD
    CdW2KlBCeqbQPcsFmWyWugxdcekhYsAWyoSf818NUsZdBWBaR/OukXrNLfkQ79Iy
    ZohZbvabO/X+MVT3rriAoKc8oE2Uws6DF+60PV7/WIPjNvXySdqspImSN78mflxD
    qwLqRBYkA3I75qppLGG9rp7UCdRjxMl8ZDBld+7yvHVgt1cVzJx9xnyGCC23Uaic
    MDSXYrB4I4WHXPGjxhZuCuPBLTdOLU8YRvMYdEvYebWHMpvwGCF6bAx3JBpIeOQ1
    wDB5y0USicV3YgYGmi+NZfhA4URSh77Yd6uuJOJENRaNVTzk
    -----END CERTIFICATE-----
  

    -----BEGIN CERTIFICATE-----
    MIIFHDCCAwSgAwIBAgIJANUP8luj8tazMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
    BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMTkxMTIyMjAzNzU4WhcNMzQxMTE4MjAz
    NzU4WjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
    AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
    Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
    tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
    nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
    C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
    oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
    JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
    sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
    igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
    RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
    aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
    AGMCAwEAAaNjMGEwHQYDVR0OBBYEFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMB8GA1Ud
    IwQYMBaAFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMA8GA1UdEwEB/wQFMAMBAf8wDgYD
    VR0PAQH/BAQDAgIEMA0GCSqGSIb3DQEBCwUAA4ICAQBOMaBc8oumXb2voc7XCWnu
    XKhBBK3e2KMGz39t7lA3XXRe2ZLLAkLM5y3J7tURkf5a1SutfdOyXAmeE6SRo83U
    h6WszodmMkxK5GM4JGrnt4pBisu5igXEydaW7qq2CdC6DOGjG+mEkN8/TA6p3cno
    L/sPyz6evdjLlSeJ8rFBH6xWyIZCbrcpYEJzXaUOEaxxXxgYz5/cTiVKN2M1G2ok
    QBUIYSY6bjEL4aUN5cfo7ogP3UvliEo3Eo0YgwuzR2v0KR6C1cZqZJSTnghIC/vA
    D32KdNQ+c3N+vl2OTsUVMC1GiWkngNx1OO1+kXW+YTnnTUOtOIswUP/Vqd5SYgAI
    mMAfY8U9/iIgkQj6T2W6FsScy94IN9fFhE1UtzmLoBIuUFsVXJMTz+Jucth+IqoW
    Fua9v1R93/k98p41pjtFX+H8DslVgfP097vju4KDlqN64xV1grw3ZLl4CiOe/A91
    oeLm2UHOq6wn3esB4r2EIQKb6jTVGu5sYCcdWpXr0AUVqcABPdgL+H7qJguBw09o
    jm6xNIrw2OocrDKsudk/okr/AwqEyPKw9WnMlQgLIKw1rODG2NvU9oR3GVGdMkUB
    ZutL8VuFkERQGt6vQ2OCw0sV47VMkuYbacK/xyZFiRcrPJPb41zgbQj9XAEyLKCH
    ex0SdDrx+tWUDqG8At2JHA==
    -----END CERTIFICATE-----
  

    -----BEGIN CERTIFICATE-----
    MIIFHDCCAwSgAwIBAgIJAMNrfES5rhgxMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
    BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMjExMTE3MjMxMDQyWhcNMzYxMTEzMjMx
    MDQyWjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
    AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
    Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
    tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
    nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
    C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
    oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
    JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
    sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
    igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
    RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
    aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
    AGMCAwEAAaNjMGEwHQYDVR0OBBYEFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMB8GA1Ud
    IwQYMBaAFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMA8GA1UdEwEB/wQFMAMBAf8wDgYD
    VR0PAQH/BAQDAgIEMA0GCSqGSIb3DQEBCwUAA4ICAQBTNNZe5cuf8oiq+jV0itTG
    zWVhSTjOBEk2FQvh11J3o3lna0o7rd8RFHnN00q4hi6TapFhh4qaw/iG6Xg+xOan
    63niLWIC5GOPFgPeYXM9+nBb3zZzC8ABypYuCusWCmt6Tn3+Pjbz3MTVhRGXuT/T
    QH4KGFY4PhvzAyXwdjTOCXID+aHud4RLcSySr0Fq/L+R8TWalvM1wJJPhyRjqRCJ
    erGtfBagiALzvhnmY7U1qFcS0NCnKjoO7oFedKdWlZz0YAfu3aGCJd4KHT0MsGiL
    Zez9WP81xYSrKMNEsDK+zK5fVzw6jA7cxmpXcARTnmAuGUeI7VVDhDzKeVOctf3a
    0qQLwC+d0+xrETZ4r2fRGNw2YEs2W8Qj6oDcfPvq9JySe7pJ6wcHnl5EZ0lwc4xH
    7Y4Dx9RA1JlfooLMw3tOdJZH0enxPXaydfAD3YifeZpFaUzicHeLzVJLt9dvGB0b
    HQLE4+EqKFgOZv2EoP686DQqbVS1u+9k0p2xbMA105TBIk7npraa8VM0fnrRKi7w
    lZKwdH+aNAyhbXRW9xsnODJ+g8eF452zvbiKKngEKirK5LGieoXBX7tZ9D1GNBH2
    Ob3bKOwwIWdEFle/YF/h6zWgdeoaNGDqVBrLr2+0DtWoiB1aDEjLWl9FmyIUyUm7
    mD/vFDkzF+wm7cyWpQpCVQ==
    -----END CERTIFICATE-----
  

    -----BEGIN CERTIFICATE-----
    MIIFHDCCAwSgAwIBAgIJAPHBcqaZ6vUdMA0GCSqGSIb3DQEBCwUAMBsxGTAXBgNV
    BAUTEGY5MjAwOWU4NTNiNmIwNDUwHhcNMjIwMzIwMTgwNzQ4WhcNNDIwMzE1MTgw
    NzQ4WjAbMRkwFwYDVQQFExBmOTIwMDllODUzYjZiMDQ1MIICIjANBgkqhkiG9w0B
    AQEFAAOCAg8AMIICCgKCAgEAr7bHgiuxpwHsK7Qui8xUFmOr75gvMsd/dTEDDJdS
    Sxtf6An7xyqpRR90PL2abxM1dEqlXnf2tqw1Ne4Xwl5jlRfdnJLmN0pTy/4lj4/7
    tv0Sk3iiKkypnEUtR6WfMgH0QZfKHM1+di+y9TFRtv6y//0rb+T+W8a9nsNL/ggj
    nar86461qO0rOs2cXjp3kOG1FEJ5MVmFmBGtnrKpa73XpXyTqRxB/M0n1n/W9nGq
    C4FSYa04T6N5RIZGBN2z2MT5IKGbFlbC8UrW0DxW7AYImQQcHtGl/m00QLVWutHQ
    oVJYnFPlXTcHYvASLu+RhhsbDmxMgJJ0mcDpvsC4PjvB+TxywElgS70vE0XmLD+O
    JtvsBslHZvPBKCOdT0MS+tgSOIfga+z1Z1g7+DVagf7quvmag8jfPioyKvxnK/Eg
    sTUVi2ghzq8wm27ud/mIM7AY2qEORR8Go3TVB4HzWQgpZrt3i5MIlCaY504LzSRi
    igHCzAPlHws+W0rB5N+er5/2pJKnfBSDiCiFAVtCLOZ7gLiMm0jhO2B6tUXHI/+M
    RPjy02i59lINMRRev56GKtcd9qO/0kUJWdZTdA2XoS82ixPvZtXQpUpuL12ab+9E
    aDK8Z4RHJYYfCT3Q5vNAXaiWQ+8PTWm2QgBR/bkwSWc+NpUFgNPN9PvQi8WEg5Um
    AGMCAwEAAaNjMGEwHQYDVR0OBBYEFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMB8GA1Ud
    IwQYMBaAFDZh4QB8iAUJUYtEbEf/GkzJ6k8SMA8GA1UdEwEB/wQFMAMBAf8wDgYD
    VR0PAQH/BAQDAgIEMA0GCSqGSIb3DQEBCwUAA4ICAQB8cMqTllHc8U+qCrOlg3H7
    174lmaCsbo/bJ0C17JEgMLb4kvrqsXZs01U3mB/qABg/1t5Pd5AORHARs1hhqGIC
    W/nKMav574f9rZN4PC2ZlufGXb7sIdJpGiO9ctRhiLuYuly10JccUZGEHpHSYM2G
    tkgYbZba6lsCPYAAP83cyDV+1aOkTf1RCp/lM0PKvmxYN10RYsK631jrleGdcdkx
    oSK//mSQbgcWnmAEZrzHoF1/0gso1HZgIn0YLzVhLSA/iXCX4QT2h3J5z3znluKG
    1nv8NQdxei2DIIhASWfu804CA96cQKTTlaae2fweqXjdN1/v2nqOhngNyz1361mF
    mr4XmaKH/ItTwOe72NI9ZcwS1lVaCvsIkTDCEXdm9rCNPAY10iTunIHFXRh+7KPz
    lHGewCq/8TOohBRn0/NNfh7uRslOSZ/xKbN9tMBtw37Z8d2vvnXq/YWdsm1+JLVw
    n6yYD/yacNJBlwpddla8eaVMjsF6nBnIgQOf9zKSe06nSTqvgwUHosgOECZJZ1Eu
    zbH4yswbt02tKtKEFhx+v+OTge/06V+jGsqTWLsfrOCNLuA8H++z+pUENmpqnnHo
    vaI47gC+TNpkgYGkkBT6B/m/U01BuOBBTzhIlMEZq9qkDWuM2cA5kW5V3FJUcfHn
    w1IdYIg2Wxg7yHcQZemFQg==
    -----END CERTIFICATE-----
  
```
如果您收到的认证链中的根证书包含此公钥，而且认证链中的所有证书均未被[撤消](https://developer.android.com/privacy-and-security/security-key-attestation?hl=zh-cn#certificate_status)，您就可以知道：

1. 您的密钥位于 Google 认为安全的硬件中；而且
2. 它具有认证证书中说明的属性。

如果认证链具有任何其他根公钥，Google 不会对硬件的安全性做出任何声明。这并不意味着您的密钥被盗，只是认证并不能证明密钥位于安全硬件中。您应该相应地调整安全假设。

如果根证书不包含此页面上的公钥，可能有两个原因：

- 最可能的原因是，设备搭载的 Android 版本低于 7.0，而且不支持硬件认证。在这种情况下，Android 提供软件实现的认证，它可以生成相同类型的认证证书，但使用的是[在 Android 源代码中硬编码](https://android.googlesource.com/platform/system/keymaster/+/android-9.0.0_r30/contexts/soft_attestation_cert.cpp#176)的密钥进行签名。由于此签名密钥不是私密的，因此认证可能是假装提供安全硬件的攻击者创建的。
- 另一个可能的原因是设备不是 Google Play 设备。在这种情况下，设备制造商可以自由创建他们自己的根证书，并按照自己的意愿就认证的意义做出声明。有关具体情况，请参阅相应设备制造商的文档。请注意，在撰写本文时，Google 未发现有任何设备制造商采用这种做法。

## 证书吊销状态列表

认证密钥可能因多种原因被吊销，包括处理不当或怀疑被攻击者提取。因此，请务必根据官方证书吊销状态列表 (CRL) 检查认证链中每个证书的状态。此列表由 Google 维护，并发布在以下网址：[https://android.googleapis.com/attestation/status](https://android.googleapis.com/attestation/status)。HTTP 响应中的 `Cache-Control` 标头决定了检查更新的频率，因此不需要为每个要验证的证书发出网络请求。该网址会返回一个 JSON 文件，其中包含任何不具有正常有效状态的证书的吊销状态。该 JSON 文件的格式遵循以下 JSON 架构（[草稿 07](https://tools.ietf.org/html/draft-handrews-json-schema-01)）定义：
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "entries": {
      "description" : "Each entry represents the status of an attestation key. The dictionary-key is the certificate serial number in lowercase hex.",
      "type": "object",
      "propertyNames": {
        "pattern": "^[a-f1-9][a-f0-9]*$"
      },
      "additionalProperties": {
        "type": "object",
        "properties": {
          "status": {
            "description": "[REQUIRED] Current status of the key.",
            "type": "string",
            "enum": ["REVOKED", "SUSPENDED"]
          },
          "expires": {
            "description": "[OPTIONAL] UTC date when certificate expires in ISO8601 format (YYYY-MM-DD). Can be used to clear expired certificates from the status list.",
            "type": "string",
            "format": "date"
          },
          "reason": {
            "description": "[OPTIONAL] Reason for the current status.",
            "type": "string",
            "enum": ["UNSPECIFIED", "KEY_COMPROMISE", "CA_COMPROMISE", "SUPERSEDED", "SOFTWARE_FLAW"]
          },
          "comment": {
            "description": "[OPTIONAL] Free form comment about the key status.",
            "type": "string",
            "maxLength": 140
          }
        },
        "required": ["status"],
        "additionalProperties": false
      }
    }
  },
  "required": ["entries"],
  "additionalProperties": false
}
```
CRL 示例：
```json
{
  "entries": {
    "2c8cdddfd5e03bfc": {
      "status": "REVOKED",
      "expires": "2020-11-13",
      "reason": "KEY_COMPROMISE",
      "comment": "Key stored on unsecure system"
    },
    "c8966fcb2fbb0d7a": {
      "status": "SUSPENDED",
      "reason": "SOFTWARE_FLAW",
      "comment": "Bug in keystore causes this key malfunction b/555555"
    }
  }
}
```


### 旧版 CRL

嵌入在旧版认证证书中的 CRL 网址继续有效。新的认证证书不会再包含 CRL 网址扩展。旧版证书的状态也包含在认证状态列表中，从而使开发者可以放心地使用该列表检查新版和旧版证书的状态。[密钥认证示例](https://github.com/google/android-key-attestation)中提供了有关如何正确验证 Android 认证密钥的示例。

