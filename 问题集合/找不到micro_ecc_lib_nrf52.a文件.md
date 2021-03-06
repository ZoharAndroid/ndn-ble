# 找不到micro_ecc_lib_nrf52.a文件？
   
* **问题描述**：
  
  * 输出的提示：
    ```cannot find ../nRF5_SDK_15.2.0_9412b96/external/micro-ecc/nrf52hf_armgcc/armgcc/micro_ecc_lib_nrf52.a: No such file or directory```：也就是提示找不到micro_ecc_lib_nrf52.a这个文件。

  * 图片显示：
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/%E9%97%AE%E9%A2%981.png)

* **解决办法和步骤**：
    1. 在Windows下安装gcc，这里需要下载[MinGW](https://mirrors.xtom.com.hk/osdn//mingw/68260/mingw-get-setup.exe)（https://mirrors.xtom.com.hk/osdn//mingw/68260/mingw-get-setup.exe），安装gcc以及make工具，具体的下载安装百度上可以直接搜索查看。注意要将MinGW加入环境变量。
    这里在安装make的时候，发现MinGW中找不到对应的make包，所以，可以通过<kbd>mingw-get install mingw32-make</kbd>来安装。

    2. 下载交叉编译工具，一开始系统是没有装这个交叉编译工具的，所以需要单独安装这个交叉编译工具。下载地址：https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads 。
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E5%B7%A5%E5%85%B7.png)
    选择Windows下的版本进行安装，建议安装到电脑的默认位置。

    3. 修改配置文件。找到nRF52840 SDK中的gcc配置文件，并进行修改。具体配置路径为：<kbd>\nRF5_SDK_15.2.0_9412b96\components\toolchain\gcc</kbd>
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BD%8D%E7%BD%AE.png)
    在该路径下，打开文件为：<kbd>Makefile.windows</kbd>，修改内容如下图（具体的修改按照下载的交叉编译工具的具体版本和路径来修改）
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

    4. 打开终端，找到相应的位置make一下。
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/2019-2-28/make%E7%BB%93%E6%9E%9C.png)

    5. 烧录成功
    ![](https://raw.githubusercontent.com/ZoharAndroid/MarkdownImages/master/2019-2-28/successful.png)
