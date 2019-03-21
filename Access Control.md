# NDN-Lite Access Control

## 概述

NDN-Lite提供应用程序支持，包括安全引导，服务发现，访问控制等。在本文档中，我们描述了在NDN-Lite中实现的访问控制协议的规范。

访问控制模块旨在：

* 提供数据机密性和访问控制，从而提高系统的安全性和隐私性。
* 供自动轻量级密钥分发和管理实现。
  
NDN物联网接入控制如下:

* 加密器和控制器之间的加密密钥（EK）协商
  
  为了获得加密器加密内容的密钥，数据生产者（在ndn-lite访问控制中称为加密器）将与系统控制器协商加密密钥（EK）。密钥将通过 Elliptic Curve Diffie-Hellman（ecdh）协商，即加密程序首先发送一个包含加密程序ecdh pub密钥位的EK请求兴趣，控制器将检查访问控制策略并回复包含控制器ecdh pub密钥位的数据包。EK将通过密钥派生函数（kdf）派生。

* 解密器和控制器之间的解密密钥（DK）分配
  
  请注意，NDN-Lite访问控制使用对称密钥进行内容加密和解密。 因此，我们有EK == DK。<br/>

  为了获得解密器解密内容的密钥，数据使用者（在ndn-lite访问控制中称为解密器）将与系统控制器协商一个临时密钥，系统控制器将使用临时密钥加密EK并将其传递给解密器。短暂密钥将通过Elliptic Curve Diffie-Hellman（ecdh）协商，即加密机首先发送一个包含解密器的ecdh pub密钥位的解密密钥请求兴趣，控制器将检查访问控制策略并回复包含控制器的ecdh pub密钥位和加密的ek的数据包。短暂密钥将通过密钥派生函数（kdf）派生，并用于解密EK。

```
EK Negotiation:

  Encryptor                                Controller
      |                                         |
      |          EK Request Interest            |
      | --------------------------------------> |
      |                                         |
      |         EK Request Response Data        |
      | <-------------------------------------- |

DK Negotiation:

  Decryptor                                Controller
      |                                         |
      |          DK Request Interest            |
      | --------------------------------------> |
      |                                         |
      |         DK Request Response Data        |
      | <-------------------------------------- |

Data Consumption:

  Decryptor                                Encryptor
      |                                         |
      |            Content Interest             |
      | --------------------------------------> |
      |                                         |
      |          Encrypted Content Data         |
      | <-------------------------------------- |
```

## 包格式 

### 1. 加密器和控制器之间的加密密钥（EK）协商

#### Interest Packet Format
```
Interest Name: /[home_prefix]/AC/<encryptor_identity>/<parameters_digest>
Parameters: {Key_Type}; {Key_ID}; {EC_Pub_Key};
Signature: Signed by producer's identity key
```

#### 参数TLV将是

```
AC_Key_Type = TLV_AC_KEY_TYPE_TYPE Length
                nonNegativeInteger
AC_Key_ID = TLV_NAME_TYPE Length
              NameValue
ECDH_Public_Key = TLV_ECDH_Pub_TYPE Length
                    Bytes
```

#### AC Key Type
```
* 0: Encryption Key (EK) (This should be selected for EK request)
* 1: Decryption Key (DK)
```

#### AC Key ID是遵循命名约定的Key的名称
```
AC Key ID = /[encryptor_prefix]/[data_prefix]/KEY/<random_number>
```

#### Data Packet Format
```
Name: /[home_prefix]/AC/<encryptor_identity>/<parameters_digest>
Content: EK Response TLV
Signature: Signed by controller's identity key
```

#### EK Response TLV
```
ECDH_Public_Key = TLV_ECDH_Pub_TYPE Length
                    Bytes
Salt = TLV_SALT_TYPE Length
         Bytes
LiftTime = TLV_AC_KEY_LIFETIME_TYPE Length
             NonNegativeInteger
```

#### 指令

收到EK请求兴趣后，控制器应该

* 检查访问控制策略，以查看加密器是否有权加密内容以及密钥生存期应该是多长
* 如果是，则控制器应首先生成ECDH公钥并获取共享密钥。
* 控制器使用KDF从共享密钥中获取可选的salt值来导出AES KEY。
* 控制器记录与AES KEY位相关的KEY ID和生成时间。
* 控制器回复ECDH公钥，密钥生存期和可选的salt值。

### 解密器和控制器之间的解密密钥(DK)协商

#### Interest Packet Format

与EK请求相同但应该有AC Key Type = 1，这意味着获取解密密钥。

#### Data Packet Format
```
Name: /[home_prefix]/AC/<decryptor_identity>/<parameters_digest>
Content: DK Response TLV
Signature: Signed by controller's identity ke
```

#### DK Response TLV
```
ECDH_Public_Key = TLV_ECDH_PUB_TYPE Length
                    Bytes
Salt = TLV_SALT_TYPE Length
         Bytes
LiftTime = TLV_AC_KEY_LIFETIME_TYPE Length
             NonNegativeInteger
Encrypted_DK = TLV_CIPHER_DK_TYPE Length
                 Bytes
```

#### 指令

收到DK请求后，控制器应该
*  检查访问控制策略以查看解密器是否有权解密内容
*  如果是，则控制器应首先生成ECDH公钥并获取共享密钥。
*   控制器使用KDF从共享密钥中获取可选的salt值来导出AES KEY。
*   控制器使用AES KEY加密所需的EK
*   控制器获得EK的寿命并计算DK的寿命`DK_lifetime = EK_generated_time + EK_liftime - current_time`
*   控制器返回ECDH公钥、密钥生存期、可选的salt值和加密的EK。

## NDN-Lite访问控制的默认设置

* ECDH Protocol: NIST p-256 Curve
* KDF: Hmac-based Key Derivation Function (HKDF)
* AES Key: 128 bits from HKDF
* AES IV: derived from HKDF
* AES Encryption: CBC mode

## 加密内容TLV的数据格式

当数据内容应该被加密时，NDN-Lite定义加密内容TLV。
```
Content = ENCRYPTED_CONTENT_TLV Length
            AC_Key_ID AES_IV
            Encrypted_Payload

AC_Key_ID = TLV_NAME_TYPE Length
                NameValue

AES_IV = TLV_AES_IV_TYPE Length
           Bytes

Encrypted_Payload = TLV_ENCRYPTED_PAYLOAD_TYPE Length
                Bytes
```

当消费者收到数据包时，消费者应该

* 检查它是否有相应的DK
* 如果是，消费者可以直接解密内容
* 如果不是，消费者应尝试使用NDN-Lite访问控制协议从控制器申请密钥。
* 在获得DK之后，消费者应该缓存解密直到密钥到期。