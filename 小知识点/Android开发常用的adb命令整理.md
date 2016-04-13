---
title: Android开发常用的adb命令整理
date: 2015-03-02 12:33:45
tags: adb
categories: 技术
---



adb的全称是Android Debug Bridge

adb start-server
启动adb服务,如果它没启动的话

adb kill-server
关闭服务

adb devices
查看所连接的设备以及设备所对应的序列号
<!--more-->
adb install -r xxxx.apk
安装app,需要注意的是如果连接了两台设备,则会报错,此时可以添加-s <serialNumber>来处理

adb unstall packagename
卸载app

adb shell pm clear packagename
清除应用的数据,很常用吧?

adb connect <device-ip-address>
连接到指定的ip,这个通常配合wifidebug

adb shell
进入shell环境

adb shell dumpsys activity top
查看栈顶Activity,可以用来获取包名

adb shell pm list packages -f
查看所有已安装的应用的包名

adb shell dumpsys activity
am的状态 Activity Manager State

adb shell dumpsys package
包信息 Package Information

adb shell dumpsys meminfo
内存使用情况Memory Usage

adb shell dumpsys procstats
Memory Use Over Time

adb shell dumpsys gfxinfo
Graphics State

adb pull <remote> <local>
从手机复制文件出来

adb push <local> <remote>
向手机发送文件

eg.  adb push foo.txt /sdcard/foo.txt

adb shell cat /proc/cpuinfo
查看手机CPU,可以看到手机架构(eg.ARMv7) 和几核处理器

adb version
查看adb版本

adb help
进入adb帮助界面

其实 am,pm 其实还有很多命令,以后有多的再写吧


<a herf="https://developer.android.com/intl/zh-cn/tools/help/adb.html">adb-官方资料</a>

