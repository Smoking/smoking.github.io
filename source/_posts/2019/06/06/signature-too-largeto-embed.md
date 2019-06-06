---
title: Codesing Command failed signature too large to embed (size limitation of on-disk representation)
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-06 13:50:32
img: https://i.loli.net/2019/06/06/5cf8aaa6cae9a32783.jpg
keywords: codesign,failed,bug,signature,large,size
summary: Codesing Command failed signature too large to embed (size limitation of on-disk representation)
categories: 编程
tags:
  - 脚本
  - macosx
---
# codesign : signature too large to embed (size limitation of on-disk representation)

今天同事新增一个模块之后codesign写的签名脚本出现这个bug，网上搜索一圈两个解决方法

## --timestamp=none

codesign后面带上--timestamp=none 参数

## --signature-size

codesign后面带上--signature-size=13666 这个值设置大一点，具体设置需要设置多大跟进签名文件大小验证