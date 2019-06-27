---
title: libwebsocket windows 编译
date: 2019-05-17 17:03:15
author: smoking
img: https://i.loli.net/2019/06/27/5d14b0633eca169746.png
top: false
cover: false
coverImg: /medias/featureimages/2.jpg
toc: false
mathjax: false
keywords: windows,c++,代码,编程,libwebsocket
summary: libwebsocket windows 编译
categories: 编程
tags:
  - C++
  - windows 
---
## 环境：win7+visual studio 2015 cmake

### 下载源码

libsocket源码：https://github.com/warmcat/libwebsockets

### 官方文档说明

官方文档说明：libwebsockets/README.build.md 中windows（VS）编译：

@section cmw Building on Windows (Visual Studio)

Install CMake 2.6 or greater:http://cmake.org/cmake/resources/software.html

Install OpenSSL binaries.http://www.openssl.org/related/binaries.html

(NOTE: Preferably in the default location to make it easier for CMake to find them)

NOTE2: Be sure that OPENSSL_CONF environment variable is defined and points at \bin\openssl.cfg

Generate the Visual studio project by opening the Visual Studio cmd prompt:

cd

md build

cd build

cmake -G "Visual Studio 10" ..

(NOTE: There is also a cmake-gui available on Windows if you prefer that)

NOTE2: See this link to find out the version number corresponding to your Visual Studio edition:http://superuser.com/a/194065

Now you should have a generated Visual Studio Solution in your/builddirectory, which can be used to build.

Some additional deps may be needed

iphlpapi.lib

psapi.lib

userenv.lib

If you're using libuv, you must make sure to compile libuv with the same multithread-dll / Mtd attributes as libwebsockets itself


### 实现步骤

第一步  安装CMake:按照提示完成就行。

第二步  安装OpenSSl：按照提示完成，主要配置好环境变量

第三步

1、按照官网提示使用VS自带的命令窗进行输入指令：

![](https://i.loli.net/2019/06/27/5d14af54977db40935.png)

2、进入指定的libwebsock源码目录创建目录

![](https://i.loli.net/2019/06/27/5d14af5483c8b13589.png)

![](https://i.loli.net/2019/06/27/5d14af547549889948.png)


cmake-G "Visual Studio 14".. -DLIB_SUFFIX=64 -DLWS_WITH_HTTP2=1     -DLWS_OPENSSL_INCLUDE_DIRS=E:\Extern_Library\openssl\inc32\openssl -DLWS_OPENSSL_LIBRARIES="E:\Extern_Library\openssl\out32\libeay32.lib"

Bulid 目录下生成libwebsocket工程


![](https://i.loli.net/2019/06/27/5d14af549d74364021.png)

vs 编译工程 ，发现找不到OpenSSL头文件

设置 OpenSSL的路径

完成




[深圳利程电子有限公司](https://www.lcptcheater.com)


------------------------------------------------
**所有图片均来自网络**