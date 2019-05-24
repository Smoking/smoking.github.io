---
title: electron 代码加密
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-05-24 15:45:39
keywords: electron,代码,加密,解密,nodejs,前端,AES
img: https://i.loli.net/2019/05/24/5ce7a7c0373c090661.jpg
summary:
categories: 编程
tags:
- electron
- 前端
---


## 常见前端代码加密方式

​	如果“加密”就是让别人无法反向工程摸索你的代码逻辑的话，那是没办法的，其实任何一种语言都避免不了被反向工程，而JavaScript的特点是下载到了客户的机器上执行，所以无论如何都可以被摸清楚，只是困难度而已！那我们怎么增加困难度呢？

常见的一些nodejs “加密的办法”

- 代码压缩
- 代码混淆
- JS 自己加载 加密的文件和执行内容（解密方法暴露了）
- 代码内容存入一些特殊的文件内（例如png等），然后加载
- 利用 enclosejs （是一款开源的打包、加密工具；用于打包、加密nodejs的的工程）
- ….

## electron 中的前端代码

​	electron的 代码是直接随安装包一起发布的，上述的一些方法完全起不到任何作用，electron又不像 nw、cef那样提供一些丰富的接口可以达到加密的目的，官方只有一个 asar 压缩代码，那怎么解决呢？

加密的目的：

- 源码加密
- 解密方式不好破解

纯前端方式无法实现，那么借助C++来实现，如何实现：

- C++ 调用读取加密文件
- C++ 解密文件
- C++ 加载运行JavaScript

这样就可以比较完整的解决加密的问题，C++ 不知道加密文件地址，那么把加密文件路径传递给C++解密去运行

## 代码加密实现

需要实现的内容：

- 遍历获取所有的 JS 代码文件
- AES 加密文件内容存放到一个地址
- 源文件内容修改为 加载C++加密模块 传入加密之后的路径

### 获取所有 JS 代码路径

```javascript
//遍历 递归 获取js文件列表
function GetJsFiles(pathDir) {
    var files = fs.readdirSync(pathDir)
    files.forEach(function(filename) {
        var filepath = path.join(pathDir, filename)
        var info = fs.statSync(filepath)
        if (info.isFile() && path.extname(filepath) == ".js") {
            // console.log(filepath)
            jsFileList.push(filepath)
        }
        if (info.isDirectory()) {
            GetJsFiles(filepath)
        }
    })
}
```

​	

### 文件内容AES 加密

```javascript
const crypto = require('crypto')
const fs = require('fs')
const path = require('path')
const md5file = require('md5-file')

//输入 utf8 输出 binary 加密
var encryptData = function(chCodeId, filedata) { //传入文件的MD5值 作为key 和向量的 参数

    var _iv = new Buffer(chCodeId.slice(4, 20)) //从md5值 的第5位开始取16个值
	var _ucKey = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
    var _Key = new Buffer(_ucKey)
    
    console.log(_iv.toString())
    var cipher = crypto.createCipheriv(algorithm, _Key, _iv)
    var crypted = cipher.update(filedata, 'utf8', 'binary')
    crypted += cipher.final('binary')
    return crypted
}
```



## 修改原JS代码文件的内容

```javascript
function modifyJsfile(filepath, md5value,isEnode) {
  var stringcode = `
  const path = require('path');
  const resourcesDirPath = (process.resourcesPath = process.resourcesPath || process.env.resourcesPath);
  var yzj_loadjs = require('decrypt');
  yzj_loadjs.Load(${isEnode},resourcesDirPath, '${md5value}' , null, require, null, __filename, __dirname, process);
  yzj_loadjs = null;
  `
  fs.writeFileSync(filepath, stringcode.trim(), 'utf8')
}
```

通过 process.resourcesPath = process.resourcesPath || process.env.resourcesPath 获取electron下资源文件夹目录

require('decrypt') 为解密模块

传入的参数：

- isEnode 是否加密了
- resourcesDirPath 资源文件目录
- md5value 源文件加密之后的md5值 用来找到源文件 

其他参数有什么用，后面解密部分你就会了解

## 加密JS文件列表 并且写入 指定目录中

```javascript
function EncodeJSfiles(fileList, encodeJSfilePath,isEnode) {
    fileList.forEach(function(filename) {
        //获取原文件的md5值
        md5file(filename, function(err, hash) {
            if (err) throw err
            var filedata = fs.readFileSync(filename, 'utf8')
            console.log('读取文件内容文件:' + filename)
            if (!fs.existsSync(encodeJSfilePath)) {
                fs.mkdirSync(encodeJSfilePath)
            }
            var savefilename = path.join(encodeJSfilePath, hash)
            console.log('对文件内容进行加密处理:' + filename)
            if (isEnode == true) {
                var enfileData = encryptData(hash, filedata)
                console.log('对加密的内容存入文件:' + savefilename)
                fs.writeFileSync(savefilename, enfileData, { 'encoding': 'binary' }) //把加密后的内容写入文件
            } else {
                console.log('对未加密的内容存入文件:' + savefilename)
                fs.writeFileSync(savefilename, filedata, { 'encoding': 'utf8' }) //把加密后的内容写入文件
            }
            console.log('对原js代码文件内容进行更改:' + filename)
            modifyJsfile(filename,hash,isEnode)
        })
    })
}
```



## 总结

代码加密部分比较容易实现，主要是解密，解密请参考下一篇文章 

---


------------------------------------------------
**所有图片均来自网络**