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

![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/SSPBasicOverview_1.png?raw=true)
左边的数字（取自原稿）演示了基本SSP的消息详细信息。请注意，所有的键控材料都被选择以128位安全级别为目标，KT由diffie-hellman（例如，ecdh）生成，双方交换n1和n2作为令牌来执行diffie-hellman（更多细节见顶部链接的签名纸）。

## 2)应用程序开发人员的登录协议接口概述

登录协议的当前实现通过控制器端（当前作为Android的ndn-lite支持库的一部分实现）和设备端（当前是ndn-lite库的一部分）上的一组API实现了登录协议基本版本的双向交换。

Android和ndn-lite的实现都遵循一种模式，即使用较低级别的登录相关API来处理和构建与登录协议相关的消息(https://github.com/named-data-iot/ndn-lite/blob/master/app-support/bootstrapping/secure-sign-on-files/secure-sign-on/variants/basic/sign-on-basic-client.h, https://github.com/peurpdapeurp/ndn-lite-android-support-library/blob/master/android_library_and_example/ndnlitesupport/src/main/java/NDNLiteSupport/SignOnBasicControllerBLE/secureSignOn/SignOnController.java)，同时更高级别的登录相关API，结合这些较低级别的API与特定传输（例如BLE）为来用户提供易于使用的API来进行引导 (https://github.com/named-data-iot/ndn-lite/blob/master/app-support/bootstrapping/secure-sign-on-files/secure-sign-on-nrf-sdk-ble/sign-on-basic-client-nrf-sdk-ble.h, https://github.com/peurpdapeurp/ndn-lite-android-support-library/blob/master/android_library_and_example/ndnlitesupport/src/main/java/NDNLiteSupport/SignOnBasicControllerBLE/secureSignOn/SignOnController.java)。应用程序开发人员只应使用与传输耦合的更高级别的登录API，并将在以下各节中对它们进行描述。

在当前的实现中，SSP实现的唯一传输是蓝牙低能耗。在Android API中，这称为SecureSignOnControllerble对象，而在设备API中，这称为sign-on-basic-client-nrf-sdk-bled对象（用于蓝牙低功耗的设备端包装器和登录协议客户端目前仅针对nRF52840板实现，使用nRF SDK为BLE通信提供的库）。因此，下面的章节将仅描述与BLE上的SSP相关的更高级别的API；但是，与其他传输上的SSP接口可能非常相似。

### a) 客户端SSP over BLE接口

为了在使用NDN-Lite库的受限设备的应用程序中使用SSP over BLE，只需要添加对程序的调用，以通过BLE单例构建和初始化登录客户端。

下图显示了可以执行的功能，以启动当前在NDN-Lite库中实现的登录客户端（对于nRF SDK）：
![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/SignOnBasicBLEClient_1.png?raw=true)

调用此函数将自动构建一个基本SSP客户端，该客户端使用nRF SDK库的BLE库作为将在其上进行登录的传输。所有与SSP相关的作业（使用适当的值初始化基本客户端对象）和传输相关作业（初始化可用堆栈，执行由控制器发现的广告）将由此函数调用处理。

用于初始化登录对象的参数的详细文档可以在本节开头链接的头文件中找到，尽管这里也会给出一个简短的摘要（按参数显示的顺序）：Basic SSP variant：这是将使用的基本SSP客户端的变体。 目前，只实现了ECC_256变体; https://github.com/named-data-iot/ndn-lite/blob/master/app-support/bootstrapping/secure-sign-on-files/secure-sign-on/variants/basic/variants/ecc_256/sign-on-basic-ecc-256-consts.h 有更多的详细。如果传入NULL，则将使用默认变体，该变体当前是基本SSP的ECC_256变体。不同基本SSP变体之间的主要区别在于用于协议内安全操作的安全性/加密类型（RSA与ECC）。设备标识符：这是一系列字节，对于每个接受SSP的设备都应该是唯一的。它用于区分为控制器提供SSP的设备。设备功能：这是一系列字节，用于对设备的功能进行编码。目前在该字段中没有位的分配; 它是为了将来使用。安全登录代码：这是控制器和设备之间的预共享密钥。它应该预先生成并手动输入SSP控制器和客户端。KS公钥：这是KS密钥对的公钥，它是SSP控制器和客户端之间的预共享非对称密钥对。它应该预先生成并手动输入SSP控制器和客户端。 密钥的格式在该部分顶部链接的头文件中描述。KS私钥：这是KS密钥对的私钥。登录完成回调：一个回调函数，用于为正在初始化的登录客户端完成登录。

### b) 控制器端SSP over BLE接口

为了在Android设备上使用NDN-Lite android支持库（可以在这里找到：https://github.com/peurpdapeurp/ndn-lite-android-support-library）的应用程序中使用SSP over BLE，只需要添加一个调用程序来构建和初始化基于BLE单例的基本SSP控制器。

下图显示了当前在NDN-Lite android支持库（适用于Android设备）中实现的BLE启动基本SSP控制器时可以执行的功能:
![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/SignOnBasicBLEController_1.png?raw=true)

在上图中注意，在通过BLE单例初始化基本SSP控制器之前，首先应该执行以下操作：初始化NDN-Lite android支持库。 初始化BLEUnicastConnectionMaintainer单例。 如果未初始化，则BLEFace对象和SignOnBasicControllerBLE单例都不会按原样运行; 有关详细信息，请参阅本文档的第三部分。

同样重要的是要注意，为了成功地为设备执行SSP协议，必须将其添加到控制器的设备列表中，这些设备预计将通过SSP进行登录。

这可以通过下图所示的函数调用来完成：
![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/SignOnBasicBLEController_2.png?raw=true)

从上图中可以看出，在初始化SignOnBasicControllerBLE单例之后，应该使用KS密钥对公钥的NDN证书，设备标识符和预期的设备的安全登录代码来调用addDevicePendingSignOn函数。通过SSP接受入职。应当注意，在上面的示例中，使用原始KS密钥对公钥来创建KS密钥对公钥证书（即，表示密钥所需的最小字节量，与micro-ecc库中使用的格式相同）。https://github.com/kmackay/micro-ecc）; 让KS公钥证书由其他一些相互信任的方签名更安全，但上面的例子主要用于演示如何使用Android上基本SSP控制器的API。

## 3)目前SSP在BLE上的实现

以下部分将描述SSP对BLE的基本传输机制; 本节主要面向有兴趣了解当前实现的基础机制的开发人员，以及可能希望为其他平台实现类似内容的开发人员。

本节将分为两部分; 第一部分将描述当前的解决方案，而第二部分将解释为什么做出某些设计选择。

### a)描述目前在BLE上实现的SSP

SSP over BLE的当前传输解决方案与NDN-Lite库的BLE face功能交织在一起; 因此，有必要简要讨论用于NDN-Lite库的当前BLE face解决方案。 关于为什么会出现这种情况的说明将在下一节中给出。

在当前的NDN-Lite BLE face实现中，BLE 4.2和BLE 5都被使用。与此设计选择相关的BLE 4.2和BLE 5的主要区别在于BLE 4.2使用“传统”广告，其中最大有效载荷为31字节(见 https://blog.bluetooth.com/exploring-bluetooth5-whats-new-in-advertising)，而BLE 5可以同时使用“传统”和“扩展”广告。在“扩展”广告中，理论上最大有效载荷要高得多，尽管实际上在nRF52840板上最大有效载荷是218字节(见https://devzone.nordicsemi.com/f/nordic-q-a/41180/maximum-amount-of-data-that-can-be-sent-through-extended-advertisements-nrf52840)，并且BLE 5在不同硬件上的实现可能有所不同。因为广告是在BLE中广播数据的唯一方式，所以这些广告包是在BLE中实现多播 face的关键部分，因为具有更大的有效负载允许广播更多的数据，而无需分段和重组(这将在下一节中详细讨论)。

在当前的解决方案中，NDN-Lite库的BLE面使用BLE单播连接与基于Android手机的控制器通信，同时使用扩展广告向其他受限设备广播数据。 板和控制器之间交换的所有信息（无论是SSP相关消息还是其他任何信息）都是通过BLE单播连接完成的，而板之间交换的信息是通过扩展广告完成的。

该系统的主要复杂性来自这样一个事实：在维持单播连接时无法进行扩展广告，至少在nRF52840板上（这已通过实验验证）。但是，同时，希望为NDN-Lite库的BLE面向用户提供隐藏这种复杂性的抽象，以便通过NLE-Lite BLE face发送的数据通过BLE单播连接发送并扩展广告（因此可以被任何Android控制器接收，该板正在维持与扩展广告范围内的BLE单播连接，以及其他板。）

在给出了上述解释之后，下面是当前实现的解决方案，用于在BLE单播连接和BLE 5的扩展广告之间共享NDN-Lite的BLE面中的BLE连接，以便允许使用NDN-Lite库的nRF52840板同时与安卓设备和其他板交换数据(发送和接收)。

#### i)NDN-Lite库的BLE Face和SSP over BLE Client

使用NDN-Lite库的设备可以选择初始化Basic SSP Client对象和/或初始化Ndn-Lite库的BLE面。 这两个对象都是单例以简化实现。 这两个对象还依赖于后端BLE实现，该实现处理诸如扫描，维护单播连接和传输（通过单播连接发送和接收信息）之类的事情。

基本SSP与BLE客户端和BLE面对面协同工作的方式基本如下：可以认为设备的“稳定”状态是与Android控制器进行单播连接。 如果它当前没有与Android控制器的单播连接，则它要么进行传统广告，以便控制器可以连接到它，如果它在范围内，或者它正在进行扩展广告。

每当通过Ndn-Lite库的BLE face发送消息时，BLE face将首先尝试通过与Android控制器的单播连接发送消息。 如果不存在单播连接，则它不会通过单播连接发送消息，但如果确实存在单播连接，则它将通过该连接发送消息。

每当通过Ndn-Lite库的BLE面发送消息时，BLE face将首先尝试通过与Android控制器的单播连接发送消息。 如果不存在单播连接，则它不会通过单播连接发送消息，但如果确实存在单播连接，则它将通过该连接发送消息。

之后，如果它被连接，NDN-Lite BLE face将从它当前所在的单播连接断开，然后通过扩展广告发送它刚刚通过单播连接发送的相同消息，以便使用NDN-Lite库的BLE面的其他板也可以检测到该消息。

在BLE面部通过扩展广告发送消息之后，它将开始传统广告; 这里的期望是控制器将主动连接到广告Ndn-Lite相关服务UUID的设备，以便设备可以返回其“稳定”状态。

连接到控制器的原因被认为是“稳定”状态，因为这是设备可以接收最多信息的状态。 如果设备处于单播连接中，它仍然可以扫描扩展广告，这意味着如果另一个设备通过扩展广告发送消息，或者Android控制器通过BLE单播连接发送消息，则设备可以同时接收这两个消息 。

但是，当设备未连接到Android控制器时，它将无法从控制器接收BLE单播消息; 因此，当设备与控制器断开连接以进行扩展广告时，它将只能检测来自其他电路板的消息进行扩展广告。 这就是为什么设备一做好扩展广告就会恢复传统广告的原因; 它希望通过控制器返回稳定状态，以便它可以与尽可能多的其他实体（其他板和Android控制器）交换数据。

#### ii)NDN-Lite Android支持库的BLE Face和SSP over BLE控制器

Controller包含一个BLEUnicastConnectionManager，它使用NDN-Lite库不断扫描设备，并在检测到它们时自动连接到它们。 如果使用NDN-Lite库的设备已通过BLE单例初始化BLE面部或其基本SSP客户端，它将自动开始通过BLE进行广告，并且可以通过使用已初始化的NDN-Lite Android支持库的Android设备进行检测 它的BLEUnicastConnectionManager单例。

使用观察者模式，SignOnControllerBLE对象和已创建的任何BLEFace对象可以观察BLEUnicastConnectionManager单例，并被告知何时建立与NDN-Lite设备的连接，并且还接受来自它们的BLE相关消息。 BLEUnicastConnectionManager和SignOnControllerBLE对象都是单例以简化实现，但BLEFace对象不是; 应为每个需要BLE连接的设备创建一个新的BLEFace对象。

当前实现的一个怪癖是，即使在为特定设备成功执行SSP协议之后，SignOnControllerBLE对象仍将通过BLEUnicastConnectionManager接收从该设备接收的所有BLE消息(所有已创建的BLEFace对象也是如此)；在当前的解决方案中，SignOnControllerBLE对象将简单地忽略这些消息，因为它将尝试解析它们以检查它们是否是有效的SSP消息，如果不是，则忽略它们。

#### iii)NDN-Lite设备的BLE状态图

![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/NDNLiteBLEStates.png?raw=true)

### b)解释当前在BLE上实现SSP的设计选择

由于目前没有Android手机支持检测BLE 5扩展广告，因此实现的许多部分都是必需的。因此，BLE单播连接和BLE 5扩展广告的组合被用于允许Android手机作为控制器和使用ndn-lite库进行通信的受限设备（如NRF52840板）。

这里的主要问题是BLE 4.2的“遗留”广告只能包含31个字节的有效载荷（我们已通过实施验证了这一点，并且在许多来源中也提到了这一点，例如这个 https://blog.bluetooth.com/exploring-bluetooth5-whats-new-in-advertising.

 对于使用BLE广告实现多播NDN面部以具有更大的广告有效载荷将是理想的; 这正是BLE 5提供的一个功能，因此这是当前在Ndn-Lite库中实现BLE面的功能，因此设备可以发送相当大量的信息（目前最多218个字节） ，请参见此处：https://devzone.nordicsemi.com/f/nordic-q-a/41180/maximum-amount-of-data-that-can-be-sent-through-extended-advertisements-nrf52840通过BLE 5扩展广告同时向多个设备广播。

 尽管可以使用碎片和重组来使用较小的传统广告有效载荷来实现BLE面部，但是一旦对BLE 5的更广泛支持可用，使用扩展广告已经实现了BLE多播面部将是有帮助的。 因此，NDN-Lite BLE面部使用扩展广告.

 但是，目前对BLE 5的支持还不够，特别是对于Android手机。 虽然有几款Android手机声称支持BLE 5和扩展广告（例如Google Pixel 2，已经进行了评估），但它们只提供部分支持。 例如，我们通过实验发现，Google Pixel 2只能发送扩展广告，但不能检测到它们。 其他几个用户也遇到过与其他手机类似的问题：https://devzone.nordicsemi.com/f/nordic-q-a/37885/extended-advertising-with-sdk-15-1-seems-not-to-work

 因此，目前控制器和设备端的实现在它们各自的实现中共享BLE面之间的BLE连接，并且在它们各自的实现中共享登录组件（即，Android侧的SignOnControllerBLE对象，以及设备端的sign-on-basic-client-nrf-sdk-ble对象）。

 以这种方式共享BLE连接允许电话和电路板在几乎所有情况下彼此通信，如前一节中所述。

