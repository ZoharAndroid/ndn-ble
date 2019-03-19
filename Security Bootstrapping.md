# Security Bootstrapping

## 概述

本文档分三部分介绍了为NDN-Lite库中的应用程序提供的安全引导支持：

1. 安全登录协议概述，这是为安全引导实现的协议。本节旨在简要概述登录协议的设计，这有助于理解底层机制，并在需要时在不同的平台上实现相同的协议。
2. 应用程序开发人员使用此功能的接口。本节旨在帮助那些使用ndn lite库开发应用程序的人。
3. 目前在NRF板上通过BLE实现该协议。本节旨在帮助那些有兴趣帮助改进实现的人，或作为那些希望更好地理解当前在ndn-lite中实现引导的基础机制的人的参考。

如果您想了解更多的有关登录协议的信息，可以在此处阅读原始论文：
[ndn-sercure-sign-on.pdf](https://github.com/ZoharAndroid/ndn-ble/blob/master/assets/ndn-secure-sign-on.pdf)

## 协议概述

