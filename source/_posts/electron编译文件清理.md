---
title: electron编译文件清理
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-05-24 15:45:29
keywords: electron,shell,清理,编译,打包
img: https://i.loli.net/2019/05/24/5ce7a7c01ddb430175.jpg
summary:
categories: 编程
tags:
- electron
- shell
---

## windows 平台清理垃圾文件

### 对执行程序无用的垃圾文件有那些

- C++工程编译的文件
- mac 端才使用的模块文件
- 文档文件 例如 markdown、doc、excel 等这类开发文档文件
- C++ 源代码文件，如 cpp、cc、h、hpp 等
- objecC 源代码文件，mm、m、swift 文件等  
- mac端才能使用的二进制文件如 .dylib、.framework、.a等文件
- 可能没有用的压缩文件 tar、7z、zip文件（可选）

### 部分脚本

```shell
# 删除一些无关平台文件
  /usr/bin/find $curdir -name "linux*" | xargs rm -rf;
  /usr/bin/find $curdir -name "darwin*" | xargs rm -rf;
  
  # 删除 visualstudio 编译之后的无用编译文件
  /usr/bin/find $curdir -name "*.tlog" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.pdb" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.exp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.obj" | xargs rm -rf;
  /usr/bin/find $curdir -name "obj" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.pch" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.idb" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.ncb" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.opt" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.plg" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.res" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.sbr" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.ilk" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.aps" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.sdf" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.temp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.dcu" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.bsc" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.ipch" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.map" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.exp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.lib" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.filters" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.vcxproj" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.props" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.targets" | xargs rm -rf;
  # 删除C++、objc源代码文件
  /usr/bin/find $curdir -name "*.cpp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.cc" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.c" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.hpp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.mm" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.h" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.h" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.lzz" | xargs rm -rf;
   # 删除编译脚本文件
  /usr/bin/find $curdir -name "*.py" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.gyp" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.gyp*" | xargs rm -rf;
  /usr/bin/find $curdir -name "lzz-gyp" | xargs rm -rf;
  # 删除一些压缩文件 如果某个类型有用就别删除
  /usr/bin/find $curdir -name "*.gz" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.rar" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.7z" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.framework" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.zip" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.dylib" | xargs rm -rf;
  # 删除一些没用的说明文件
  /usr/bin/find $curdir -name "*.txt" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.md" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.doc" | xargs rm -rf;
  /usr/bin/find $curdir -name "*.docx" | xargs rm -rf;
  # 删除其他
  /usr/bin/find $curdir -name "yarn" | xargs rm -rf;




```

### mac平台

### 对执行程序无用的垃圾文件有那些

- C++工程编译的文件
- window 端才使用的模块
- 文档文件 例如 markdown、doc、excel 等这类开发文档文件
- C++ 源代码文件，如 cpp、cc、h、hpp 等
- objecC 源代码文件，mm、m、swift 文件等
- windows端才能使用的二进制文件如 dll,lib等文件
- 可能没有用的压缩文件 tar、7z、zip文件（可选）
- …..

### 部分脚本

```shell
#删除无关平台的一些文件
find $curdir -name "win32*"  | xargs rm -rf;
find $curdir -name "linux*"  | xargs rm -rf;
#mac端删除xcode编译多出来的垃圾文件
find $curdir -name "*.tlog" | xargs rm -rf;
find $curdir -name "obj" | xargs rm -rf;
find $curdir -name "*.pdb" | xargs rm -rf;
find $curdir -name "*.exp" | xargs rm -rf;
find $curdir -name "*.obj" | xargs rm -rf;
find $curdir -name "*.pch" | xargs rm -rf;
find $curdir -name "*.idb" | xargs rm -rf;
find $curdir -name "*.ncb" | xargs rm -rf;
find $curdir -name "*.opt" | xargs rm -rf;
find $curdir -name "*.plg" | xargs rm -rf;
find $curdir -name "*.tlog" | xargs rm -rf;
find $curdir -name "*.res" | xargs rm -rf;
find $curdir -name "*.sbr" | xargs rm -rf;
find $curdir -name "*.ilk" | xargs rm -rf;
find $curdir -name "*.aps" | xargs rm -rf;
find $curdir -name "*.sdf" | xargs rm -rf;
find $curdir -name "*.temp" | xargs rm -rf;
find $curdir -name "*.dcu" | xargs rm -rf;
find $curdir -name "*.bsc" | xargs rm -rf;
find $curdir -name "*.ipch" | xargs rm -rf;
find $curdir -name "*.xcodeproj" | xargs rm -rf;
find $curdir -name "*.vcxproj" | xargs rm -rf;
find $curdir -name "*.mk" | xargs rm -rf;
find $curdir -name "*.Makefile" | xargs rm -rf;
find $curdir -name "*.rc" | xargs rm -rf;
# 删除C++、objc源代码文件
find $curdir -name "*.cpp" | xargs rm -rf;
find $curdir -name "*.cc" | xargs rm -rf;
find $curdir -name "*.c" | xargs rm -rf;
find $curdir -name "*.hpp" | xargs rm -rf;
find $curdir -name "*.mm" | xargs rm -rf;
find $curdir -name "*.m" | xargs rm -rf;
find $curdir -name "*.xib" | xargs rm -rf;
find $curdir -name "*.h" | xargs rm -rf;
find $curdir -name "*.h" | xargs rm -rf;
find $curdir -name "*.lzz" | xargs rm -rf;
  # 删除编译脚本文件
find $curdir -name "*.py" | xargs rm -rf;
find $curdir -name "*.gyp" | xargs rm -rf;
find $curdir -name "*.gyp*" | xargs rm -rf;
find $curdir -name "lzz-gyp" | xargs rm -rf;
find $curdir -name "*.map" | xargs rm -rf;
find $curdir -name "*.a" | xargs rm -rf;
# 删除一些压缩文件 如果某个类型有用就别删除
find $curdir -name "*.gz" | xargs rm -rf;
find $curdir -name "*.rar" | xargs rm -rf;
find $curdir -name "*.7z" | xargs rm -rf;
find $curdir -name "*.dll" | xargs rm -rf;
# 删除一些没用的说明文件
find $curdir -name "*.txt" | xargs rm -rf;
find $curdir -name "*.md" | xargs rm -rf;
find $curdir -name "*.doc" | xargs rm -rf;
find $curdir -name "*.docx" | xargs rm -rf;
 # 删除其他
find $curdir -name "yarn" | xargs rm -rf;
```





## 总结

这只是一部分，其中还有需要没用的文件，慢慢优化减少electron安装包的体积

--- 


------------------------------------------------
**所有图片均来自网络**