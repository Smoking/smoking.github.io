---
title: electron工程打包
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-05-24 15:44:33
keywords: electron,shell,加密,解密,nodejs,c++,打包,安装包
img: https://cdn-images-1.medium.com/max/2600/1*aBsgPiEeOE5lLoippRm7BA.png
summary:
categories: 编程
tags:
- 前端
- electron
---




## 说明

这一套方案只是介绍了我在工程中用的一种解决方案，当然还有很多其他解决方案。

## 实现目的

把electron工程打包成一个安装包，在客户机器上安装完成之后可以直接使用，安装包中包含electron的运行环境，以及工程实现代码



## 完整流程



### 前端代码打包

web端代码编译、压缩、混淆代码。这个方案就不细说了，网上可以找到很多相关资料



### electron-builder 生成绿色包

需要使用 electron-builder 模块来打包electron，electron-builder 具体使用细节可以参考[官方文档](<https://www.electron.build/>),配置好基本参数例如：图标、是否压缩、源码目录等

在工程package.json中部分配置如下：

```json
"build": {
        "directories": {
            "buildResources": "resource",
            "output": "dist"
        },
        "asar": false,
        "asarUnpack": [
            "*******"
        ],
        "compression": "maximum",
        "files": [
            "build",
            "resource"
        ],
        "forceCodeSigning": false,
        "mac": {
            "icon": "resource/mac/icon.icns",
            "compression": "maximum",
            "target": [
                "dir"
            ]
        },
        "win": {
            "icon": "resource/windows/icon.ico",
            "target": [
                "dir"
            ]
        }
    }
```

你也可以使用electron-package等模块完成类似工作

#### 注意

electron可以加载asar压缩模块内容，但是有几中文件是不能放入asar中的

- C++的 dll 库文件，在asar中无法运行，因为C++没办法加载asar中文件
- C++中加载的资源文件，同上
- objc 加载的资源文件，同上

最终electron 的资源resource 目录下可以变成

- electron.asar  electron本身自动的代码
- app.asar      可以压缩的纯前端代码以及.node文件
- app.asar.unpacked    如果模块中涉及到C++ dll,那么把整个模块移入到 app.asar.unpacked，当然也可以通过动态加载C++ dll 不需要全部移到这里



### 清理nodejs C++模块中无用文件，减少总体积

​	具体实现参考[electron编译文件清理](./electron-bian-yi-wen-jian-qing-li.html) 文章



### 拷贝相关license

​	如果集成了第三方库或者功能，拷贝相应的license到 绿色包目录下 



### 拷贝C++动态库

​	目的是把包含C++ dll 或者dylib、framework 的模块 拷贝到  app.asar.unpacked 中，在windows和mac 下要注意不同的 资源路径



### 代码加密

[electron代码加密](./electron-dai-ma-jia-mi.html)

[electron代码解密](./electron-dai-ma-jie-mi.html)

### 代码签名

[windows数字签名](./windows-dai-ma-qian-ming.html)

[macosx数字签名](./macosx-qian-ming.html)


### 压缩app目录为asar文件

```shell
# windows 实现

asar pack ./dist/win-ia32-unpacked/resources/app ./dist/win-ia32-unpacked/resources/app.asar;

# mac 实现 
asar pack ./dist/mac/$AppProductName.app/Contents/Resources/app ./dist/mac/$AppProductName.app/Contents/Resources/app.asar
```



### 把绿色包制作成安装包

在npm 上有很多相关的模块

windows 可以用 nsis、innosetup 脚本来定制化你的界面

mac 上有生成dmg、pkg的模块





------------------------------------------------
**所有图片均来自网络**