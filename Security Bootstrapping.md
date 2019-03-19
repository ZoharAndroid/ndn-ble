# Security Bootstrapping

## 概述

本文档分三部分介绍了为NDN-Lite库中的应用程序提供的安全引导支持：

1. 安全登录协议概述，这是为安全引导实现的协议。本节旨在简要概述登录协议的设计，这有助于理解底层机制，并在需要时在不同的平台上实现相同的协议。
2. 应用程序开发人员使用此功能的接口。本节旨在帮助那些使用ndn lite库开发应用程序的人。
3. 目前在NRF板上通过BLE实现该协议。本节旨在帮助那些有兴趣帮助改进实现的人，或作为那些希望更好地理解当前在ndn-lite中实现引导的基础机制的人的参考。

如果您想了解更多的有关登录协议的信息，可以在此处阅读原始论文：
[ndn-sercure-sign-on.pdf](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/ndn-secure-sign-on.pdf)

## 1)协议概述

安全登录协议（简称SSP）的一般目标是允许受约束的设备通过本地方式自动安全地从可信控制器检索信任锚证书以及锚签名证书（即，不依赖于云或任何其他第三方信任中心）。它有望用于家庭物联网场景，用户可以将智能手机用作控制器，并通过登录协议将加密凭证分发给家中的设备。

在运行该协议之前，我们假设该设备在某个地方编码了一段机密信息（如二维码或条形码），并且已经通过一些带外操作与控制器共享了该信息。此信息用于建立初始信任，因此共享方法应该是安全的。这是类似协议采用的一种常见策略。但是，ssp为系统提供了更强的保护，以防这一预先共享的信息不知何故被泄露（有关更多详细信息，请参阅顶部链接的签名纸）。

![](SSPBasicOverview_1.png)
左边的数字（取自原稿）演示了基本SSP的消息详细信息。请注意，所有的键控材料都被选择以128位安全级别为目标，KT由diffie-hellman（例如，ecdh）生成，双方交换n1和n2作为令牌来执行diffie-hellman（更多细节见顶部链接的签名纸）。

## 2)应用程序开发人员的登录协议接口概述

登录协议的当前实现通过控制器端（当前作为Android的ndn-lite支持库的一部分实现）和设备端（当前是ndn-lite库的一部分）上的一组API实现了登录协议基本版本的双向交换。

Android和ndn-lite的实现都遵循一种模式，即使用较低级别的登录相关API来处理和构建与登录协议相关的消息(https://github.com/named-data-iot/ndn-lite/blob/master/app-support/bootstrapping/secure-sign-on-files/secure-sign-on/variants/basic/sign-on-basic-client.h, https://github.com/peurpdapeurp/ndn-lite-android-support-library/blob/master/android_library_and_example/ndnlitesupport/src/main/java/NDNLiteSupport/SignOnBasicControllerBLE/secureSignOn/SignOnController.java)，同时更高级别的登录相关API，结合这些较低级别的API与特定传输（例如BLE）为来用户提供易于使用的API来进行引导 (https://github.com/named-data-iot/ndn-lite/blob/master/app-support/bootstrapping/secure-sign-on-files/secure-sign-on-nrf-sdk-ble/sign-on-basic-client-nrf-sdk-ble.h, https://github.com/peurpdapeurp/ndn-lite-android-support-library/blob/master/android_library_and_example/ndnlitesupport/src/main/java/NDNLiteSupport/SignOnBasicControllerBLE/secureSignOn/SignOnController.java)。应用程序开发人员只应使用与传输耦合的更高级别的登录API，并将在以下各节中对它们进行描述。

在当前的实现中，SSP实现的唯一传输是蓝牙低能耗。在Android API中，这称为SecureSignOnControllerble对象，而在设备API中，这称为sign-on-basic-client-nrf-sdk-bled对象（用于蓝牙低功耗的设备端包装器和登录协议客户端目前仅针对nRF52840板实现，使用nRF SDK为BLE通信提供的库）。因此，下面的章节将仅描述与BLE上的SSP相关的更高级别的API；但是，与其他传输上的SSP接口可能非常相似。