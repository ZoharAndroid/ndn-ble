# ndn-lite

## 概述

ndn-lite库实现了NDN(Named Data Networking:命名数据网络)栈，ndn-lite库具有针对物联网(IoT)场景的高级应用程序支持和低级OS或者硬件的适配。

## 体系结构

ndn-lite库独立于OS和开发工具，通过平台和ndn-lite之间创建精简的适配层，公开的API可以让库很容易的插入IoT软件开发工具包（SDK）和IoT操作系统中。

ndn-lite库旨在提供核心的NDN网络栈。ndn-lite库允许应用程序直接集成一些功能，包括：访问控制、服务发现、Schematized Trust等等。

项目的体系结构如下图所示


## 参考资料

1. https://github.com/named-data-iot/ndn-lite