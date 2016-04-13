---
title: Android源码编译和内核编译
date: 2016-02-21 14:33:45
tags: 源码
categories: 技术
---

## 源码编译

    
####	 1、系统： Ubuntu14.04以上 或者 Mac
>下面的介绍都是在ubuntu下进行的
	
#### 2、JdK安装 
	 
	$ sudo add-apt-repository "deb http://archive.canonical.com/ lucid partner"
	$ sudo apt-get update
	$ sudo apt-get install sun-java8-jdk
<!--more--> 
####    3、必备软件包安装

	$ sudo apt-get install git-core gnupg flex bison gperf build-essential \
	  zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs \
	  x11proto-core-dev libx11-dev lib32readline5-dev lib32z-dev \
	  libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown \
	  libxml2-utils
#### ４　下载repo工具
    
    $ cd ~
    $ mkdir ~/bin
    $ PATH=~/bin:$PATH
    $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
    $ chmod a+x ~/bin/repo
    
#### ５　下载源码

>注意，下面最后一条命令，-b 后面的 Gingerbread 可以替换成 IceCreamSandwich 或者是 Froyo 中的任何一个。当然，还有其它选择，这个名字，就是 Android 版本的英文名，http://source.android.com/source/build-numbers.html 里面有所有的名字
   
    $ mkdir CMROM
    $ cd CMROM
    $ repo init -u git://github.com/CyanogenMod/android.git
    
>如果是使用Google官方的源码编译
    
    $ repo init -u https://android.googlesource.com/platform/manifest -b [Gingerbread]
    
    [?]部分是Android的版本名称--详见http://source.android.com/source/build-numbers.html
* 注意事项

>默认情况下，访问Android源码是匿名的，为了防止下载服务器压力过大，下载服务器对每个ip都有下载限制。如果和别人共享一个公网IP(和别人共享路由器时，便是如此),Android源码服务器便会阻止多人同时下载，容易报错。为了解决该问题，需要使用带授权的访问，源码服务器此时对用户进行限制，而不是对ip进行限制。方法如下：
先创建密码：https://android.googlesource.com/new-password
该地址也经常无法访问，需多次尝试(可相隔几分钟)，若使用代理，就没法获得有用的密钥
再将密码保存在~/.netrc里
然后强制使用带授权的访问：
 $repo init -u https://android.googlesource.com/a/platform/manifest
在国内用repo初始化时，会经常遇到101的错误，因为有墙的原因，重试多次，运气好时便可以完成，设置代理的话会更顺利一点

* 源代码的目录结构

    在讲述Android源码编译的三个步骤之前，将先介绍Android源码目录结构，以便读者理清Android编译系统核心代码在Android源代码的位置。
Android源代码顶层目录结构如下所示：

 ../CyanogenMod/
        ├──abi#应用二进制接口，不同的操作系统，应用二进制接口不同，因此linux上的二进制可执行文件在windows上无法执行
        ├──android#存放了一些xml文件，用于描述工程路径及其对应的远程仓库地址，repo工具将使用这些信息同步代码
        ├──bionic#bionic C库,Android没有使用标准的glibc库，而是自己重新实现了一套c/C++库，包括libc libdl libm libstdc++ libthread_db
        ├──bootable# 包含两个工程，recovery和diskinstaller，刷机或者系统升级都是由recovery完成的，
        ├──build#Android编译系统核心代码都存放在该目录，我们也将对该目录下的文件做详细分析
        ├──cts#Android兼容性测试套件标准
        ├──dalvik#dalvik JAVA虚拟机，Android用的java虚拟机和pc上用的jvm不一样
        ├──development#应用程序开发工具 有eclipse开发用的formatter配置
        ├──device#设备相关配置文件，存放规则 device/$vendor/$product
        ├──docs#网站文档
        ├──external#用到的第三方库 象busybox bash openssl等工具都存放在该目录
        ├──filelist#使用godir命令生成的索引文件
        ├──frameworks#核心框架——java及C++语言，可生成framework.jar
        ├──gdk#glass开发Sdk
        ├──hardware#部分厂家开源的硬件适配层HAL代码
        ├──kernel#内核源码目录 存放规则kernel/$vendor/$product
        ├──libcore#一些有用的库 像xml Jason luni
        ├──libnativehelper#Support functions for Android’s class libraries
        ├──Makefile#在顶层目录编译，利用的默认Makefile，它只是简单包含了build/core/main.mk
        ├──ndk#ndk开发工具
        ├──packages#Android apk程序所在目录,象settings，gallery等程序
        ├──pdk#Platform Development Kit The goal of the PDK release is to help chipset vendors and OEMs to migrate to a new relelase
        ├──prebuilt#x86和arm架构下预编译的一些资源
        ├──prebuilts#有clang eclipse gcc misc ndk qemu-kernel sdk tools等子目录，交叉编译工具链所在目录
        ├──sdk#sdk及模拟器
        ├──system#核心代码，包含了最小化可启动的环境，还有底层调试及检查工具，adbd也在system/core目录
        ├──tools#有子目录build和motodev，可能跟摩托罗拉有关
        ├──vendor#设备制造商专用的配置存放目录，存放规则vendor/$vendor/$product，cm编写的apk也放在该目录
build子目录存放编译系统的核心代码，包含着138个makefile，15个shell脚本，19个python脚本，7个C文件，7个C++文件，16个头文件，因此如果想分析编译系统核心代码，使用的IDE需支持这些编程语言，推荐使用eclipse，安装一些插件就可以很方便地查看这些代码
build子文件夹的目录结构如下所示：
build/
        ├── buildspec.mk.default#buildspec的模版文件，可定义一些变量比如TARGET_BUILD_VARIANT:=user，TARGET_BUILD_TYPE:=release
        ├── CleanSpec.mk#增量编译时，会执行该文件里的命令，这些命令一般用于清除中间文件
        ├── core#编译系统的核心文件放在该目录，主要是一些makefile
        ├── envsetup.sh#编译时需先用source envsetup.sh设置好环境变量，该脚本提供了许多有用的命令，比如cout,croot,cgrep,在详细介绍Android编译步骤时会列出来
        ├── libs#是一个C++模块，编译后可生成libhost.a静态库，里面的函数主要用于与编译主机交互
        ├── target#包含编译目标相关的makefile，它有两个子文件夹 board和product，产品都在该目录下定义，比如generic,full产品，定义设备产品时，会从这里继承产品
        └── tools#各种工具，多数使用python编写，工具有用于签名的signpak, 用于下载device配置的roomservice.py等，后续将详细介绍
我们在阅读build核心代码时，可能最头疼的就是变量，编译系统里有成百上千的变量，我们常常不知道其含义，容易一头雾水，为此我做了一个编译系统的参考手册供大家查阅， 可以很方便地检索变量，查看变量的意义，并有示例值。链接：http://android.cloudchou.com/


#### ６　同步源码
>别看只有一条命令，但是，下载的时间，很长的，推荐这条命令，晚上的时候，挂机执行，第二天早上，差不多能下载完。毕竟是 3GB 多的东西呢。

    $ repo sync -j 10

* 注意事项
>在工作目录里使用repo sync同步代码，期间可能会多次卡死，需要ctrl+z，然后杀掉进程，然后再次使用repo sync，因为其支持断点续传，不需要担心会从头开始下载 还可以开启多个进程同时下载，使用repo sync -j4
j4代表开启4个线程,建议i5以上的开4,i7开8

#### 4.编译源代码

* 初始化编译环境

      $source build/envsetup.sh
    
    
* 选择一个目标设备，以cm下编译htc one为例

        $lunch cm_m7ul-eng

* 此时会从网站下载m7ul的device配置以及内核源代码
所有目标设备的格式为BUILD-BUILDTYPE， BUILD是选择的目标设备，比如cm_m7,而BUILD_TYPE是eng，user或者userdebug

        user: 适合发布产品时使用，访问受限
        userdebug: 和user类型类似，有root权限和调试能力，适合调试
        eng: 开发配置，有额外的调试工具
* 编译源代码：

        $mka

## 内核编译

#### １　下载内核源码
　　每一个 Android 手机厂商，都会在自己的网站上公布已经生产的手机的内核源码，大家去小米手机的官方网站下载即可。
#### ２　解压内核源码包
　　内核源码，一般是以压缩包的形式提供下载的。大家下载之后，在 Ubuntu 中解压即可，因为命令非常简单，这里不提供。
#### ３　配置交叉工具链（编译器）路径
　　学过编程的人都知道，源码要编译成程序，必须要用编译器编译。而且手机的 CPU 不同于 PC 机，所以，得用专用的工具，即“交叉工具链”。
　　打开内核源码目录下的 Makefile 文件，找到“CROSS_COMPILE”定义的地方，确保和下面一模一样。

    CROSS_COMPILE := $(shell if [ -f .cross_compile ]; then \
        cat .cross_compile; \
        fi)

　　接着，在内核源码目录下面，新建一个叫“.cross_compile”的文件，在文件中，输入以下分隔线中间的一行内容

    /（这前面，是 Android 源码的绝对路径，要求您有一点 Ubuntu 使用经验，否则不能完成）/ANDROID_SRC/prebuilt/linux-x86/toolchain/arm-eabi-4.4.0/bin/arm-eabi-

　　打开已经下载的 Android 源码（前面有提到如何下载）目录，在 ANDROID_SRC/prebuilt/linux-x86/toolchain 下面，你会发现有几个工具链文件夹，名字为别是 arm-eabi-4.2.1（4.3.1，4.4.0，4.4.3），如果，你是用 64位 系统，用哪个都可以。如果是 32位 系统，貌似用 4.4.3 会出错。建议用 4.4.0。
　注：ANDROID_SRC 是指您电脑上，Android 源码存储的目录，请根据实际情况，进行替换。

#### ４　“.config”配置文件

　　在内核源码的目录下面，一定要有一个名字为“.config”的文件，这个文件，是小米手机的内核配置。如果内核源码的根目录下，没有发现这个文件，或者，发现的文件不是针对小米手机配置的，在 LINUX_SRC/arch/arm/configs 下面，找到针对小米手机的配置文件，复制到内核源码根目录，即可。
　注：LINUX_SRC 是指您电脑上，内核源码存储的目录，请根据实际情况，进行替换。

#### ５　选择内核配置选项

    $ cd LINUX_SRC
    $ make config

这里的config文件,是在内核源码根目录下的 config文件,如果没有直接名称为config文件的,看看是否有前缀,例如:menuconfig
　　输入上面的命令后，会出现一个怪怪的列表，通过按“空格键”，进行“选择”或“取消”某些选项。完成后，记得保存。
　注：在手机使用中，有的用户总抱怨不能使用“Wifi Tether”，不能“使用笔记本共享出来的宽带上网”，都是因为这一步，内核的选项没有被正确设置。读完这篇文章之后，大家可以自己动手了。
#### ６　“最后一步”

    $ cd LINUX_SRC
    $ make

　编译完成之后，会在“kernel/goldfish/arch/arm/boot/”目录下生成名为zImage的文件。 


## 用到的命令
#### 下载Android源码简要流程

    a. 获取repo文件: curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo .注意执行该文件需要python2.5以上版本,如果是2.4.3的python版本就无法执行这个文件脚本;
    b. 修改repo权限 : chmod a+x ~/bin/repo , 如果repo没有执行权限, 该脚本也无法执行;
    c. 初始化repo文件 : repo init -u https://android.googlesource.com/platform/manifest -b android-2.3.3_r1 , 这里要下载2.3.3版本的源码;
    d. 开始下载 :repo sync , 执行该命令就可以开始下载Android源码;

#### 下载Android内核源码简要流程

    a. 使用git下载 : git clone https://android.googlesource.com/kernel/goldfish.git ;
    b. 查看分支 : git branch -a ;
    c. 检出版本 : git checkout remotes/origin/android-goldfish-2.6.29 ;
