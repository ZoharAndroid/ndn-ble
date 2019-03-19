# NDN-BLE-nRF52840-Android

这是一个应用示例，展示了使用ndn-lite在Android手机和开发板之间进行ndn通信、安全登录和信任策略切换的基本功能。具体来说，这个应用程序由两部分组成：Android手机中的用户应用程序和开发板中的ndn-lite应用程序。**用户应用程序**是一个通用的Android应用程序，它在可用设备、基本设备信息和turst策略选项等方面提供用户界面。**ndn-lite应用程序**使用ndn-lite来提供基于ndn的通信、安全登录和信任策略切换功能等。

目前，该应用程序使用BLE作为面在Android手机和开发板之间传输数据包。

## nRF52840部分

将nRF52840中的代码分别下载到两块板子中。

因为这里是用两块板子来做实验。

## NDN-IoT-Android部分

* 需要注意的部分就是：
点手机屏幕上显示的设备的图片，可以看到信任策略选项：“仅控制器”或“所有节点”。"仅控制器"意味着用户不能通过按下按钮1或从另一个板发送命令来打开LED 1。"所有节点"指相反。按所需选项，手机将向设备发送信任策略切换兴趣，设备将闪烁LED 4指示切换。

## ndn-lite介绍

[ndn-lite]()

## 实现效果

1. 板子一上电，LED3就会闪烁3次，这表示的板子正在进行初始化相关的工作。
2. 第二次闪烁，表示的是设备在手机上已经完成了安全的sign-on。
3. 可以按按钮1去关闭LED1，或者去按按钮2去关闭LED1。
4. 如果有两块板子你可以按按钮3去发送命令兴趣包去打开另一块板子的LED1。
5. 在Android手机上，如果你点击板子图片，如果你选择"Only controller"选项，效果4就是没有用，也就是说按按钮3，是打不开另外一块板子的LED1.


## 问题集合

1. [找不到micro_ecc_lib_nrf52.a文件?]()


## 参考内容

1. [SEGGER开发IDE下载链接](https://www.segger.com/downloads/embedded-studio)：开发的IDE。
2. [nRF5 SDK下载链接](https://developer.nordicsemi.com/)：下载对应的SDK和例子。
3. [SEGGER Jlink下载](https://www.segger.com/downloads/jlink/)：jlink软件下载
4. [android-nRF-Connect apk和源代码下载链接](https://github.com/NordicSemiconductor/Android-nRF-Connect/releases)：测试ble的app
5. [nRFSniffer 下载工具](https://www.nordicsemi.com/Software-and-Tools/Development-Tools/nRF-Sniffer/Download#infotabs)：和wireshark一起使用进行ble抓包分析
6. [nRF Command-Line软件下载链接](https://www.nordicsemi.com/Software-and-Tools/Development-Tools/nRF5-Command-Line-Tools/Download#infotabs)

## 具体操作例子
1. [ndn-lite-nRF52840](https://github.com/gujianxiao/ndn-lite-application-for-nRF52840-BLE_version)：github上的Segger上面的代码，运行在nRF52840板子上面。
2. [NDN-IoT-Android](https://github.com/gujianxiao/NDN-IoT-Android)：github上面的源代码，运行在android手机上面的应用源代码