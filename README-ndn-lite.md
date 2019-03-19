# ndn-lite

## 概述

ndn-lite库实现了NDN(Named Data Networking:命名数据网络)栈，ndn-lite库具有针对物联网(IoT)场景的高级应用程序支持和低级OS或者硬件的适配。

## 体系结构

ndn-lite库独立于OS和开发工具，通过平台和ndn-lite之间创建精简的适配层，公开的API可以让库很容易的插入IoT软件开发工具包（SDK）和IoT操作系统中。

ndn-lite库旨在提供核心的NDN网络栈。ndn-lite库允许应用程序直接集成一些功能，包括：访问控制、服务发现、Schematized Trust等等。

项目的体系结构如下图所示：
![](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/iot-framework.jpg?raw=true)


## 特点

当前的ndn-lite实现了一下特点：

1. NDN层：
   * NDN编码和解码，兼容NDN TLV 0.3格式。
   * NDN IoT Forwarder，一个轻量级的IoT转发模块实现。以后会实现内容存储。
   * 可由OS/SDK特定的自适应继承抽象Face。
   * 支持单线程应用的应用-转发通信的DirectFace。
   * 仅用于测试的DummyFace。
   * 碎片：重用ndn-riot碎片头（3字节头）
  
    ```
        0           1          2           3 -->(3个字节)
        0 1 2  3    8         15           23
        +-+-+--+----+----------------------+
        |1|X|MF|Seq#|    Identification    |
        +-+-+--+----+----------------------+
    
        First bit: header bit, always 1 (indicating the fragmentation header)
        Second bit: reserved, always 0
        Third bit: MF bit, 1 indicating the last frame
        4th to 8th bit: sequence number (5 bits, encoding up to 31)
        9th to 24th bit: identification (2-byte random number)
    ```

2. 安全：
   * 加密前端，支持OS/SDK特定的加密后端实现。
   * 使用tinycrypto和micro ecc的默认纯软件加密后端。
   * 兴趣和数据签名和验证。
   * 用于数据包的AES加密内容为TLV。

3. 应用支持层：
   * 易于使用的安全引导模块，可实现高效，安全的信任锚安装和身份证书颁发。[此处查看协议详细信息](https://github.com/named-data-iot/ndn-lite/wiki/Security-Bootstrapping)。
   * 轻量级基于名称访问控制，提供数据机密性和对数据访问的控制。[此处查看协议详细信息](https://github.com/named-data-iot/ndn-lite/wiki/Access-Control)。
   * 轻量级服务发现协议模块，用于使应用程序向网络提供服务或利用网络系统中的现有服务。 [此处查看协议详细信息](https://github.com/named-data-iot/ndn-lite/wiki/Service-Discovery)。

4. 平台适配
   * Nordic NRF 802154原始驱动程序适配，包括适配层和face实现，称为ndn-nrf-802154-face。
   * Nordic SDK适配，包括适应层和face实现，称为ndn-nrf-ble-face。

## 代码基本结构

* ./encode目录：NDN包的编码和解码。
* ./forwarder目录：NDN轻量级转发实现和网络Face抽象。
* ./face目录：网络face和应用face的实现。每个face实例可能需要硬件/OS适配的支持。
* ./app-support目录：访问控制，服务发现和其他可以促进应用程序开发的高级模块。
* ./adaptation目录：硬件/OS适配。使用NDN-Lite时，开发人员应该为他们用于应用程序开发的平台/操作系统选择一个或多个适配。

## 其他说明：

* `ndn-nrf-ble-face`是Nordic SDK的适配：在`./adaptation/ndn-nrf-ble-adaptation/`下适配层。`./face/ndn-nrf-ble-face.c`和`./face/ndn-nrf-ble-face.h`是face实现。
* NRF 802154驱动程序使用`ndn-nrf-802154-face`进行适配：`./adaptation/ndn-nrf-802154-driver/`下的文件是适配层和文件`./face/ndn-nrf-802154-face.c`和`./face/ndn-nrf-802154-face.h`是face的实现


## 参考资料

1. https://github.com/named-data-iot/ndn-lite