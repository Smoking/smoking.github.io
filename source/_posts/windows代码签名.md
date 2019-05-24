---
title: windows 数字签名
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
keywords: electron,windows,签名,数字签名,代码签名,signtool
date: 2019-05-24 15:46:01
img: https://i.loli.net/2019/05/24/5ce7a8b7e828925086.jpg
summary:
categories: 编程
tags:
- windows
- shell
---



## 获取数字签名证书

​	这个不具体分析，网上可以找到很多文章怎么获取数字签名证书

## 签名工具  signtool.exe

```
signtool [command] [options] [file_name | ...]  
```

全部指令详解参考 [文档](https://docs.microsoft.com/en-us/dotnet/framework/tools/signtool-exe)

其中：

(1) **/v**：显示详细的签名结果；

(2) **/f xx.pfx**：加载代码签名证书。请把颁发给你的用户证书放到signtool目录下，或者指定文件路径；

(3) **/p 密码**：申请证书时候设置的密码；

(4) **/t,/tr**：为代码加上WoSign免费时间戳，确保签名后的代码永不过期；

(5) **test.dll**: 就是您要签名的Windows文件，如：.cab, .dll, .exe 等文件；

## 时间戳服务地址

- SHA1签名算法与时间戳 SHA-1 timestamping 

  URL： http://timestamp.verisign.com/scripts/timstamp.dll (The timstamp.dll filename is required to conform to old MS-DOS naming convention).   

- SHA-1签名算法与RFC3161 时间戳 SHA-1 with RFC 3161 timestamping 

  URL：  http://sha1timestamp.ws.symantec.com/sha1/timestamp   

- SHA-256签名算法与RFC3161 时间戳SHA-256 with RFC 3161 timestamping 

  URL： http://sha256timestamp.ws.symantec.com/sha256/timestamp

# bat脚本

```bash
echo off
:: 设置signtool.exe 目录
set curdir=%cd%\publish\win\signtool

echo %curdir%

::指定electron-builder生成的绿色包目录
set DIR="%cd%\dist\win-ia32-unpacked\"
echo DIR=%DIR%

:: SignatureSecretKeyPath 签名证书路径的环境变量，因为证书一般不放入代码工程，所以用环境变量
:: 循环遍历目录下的需要签名文件
:: password 表示证书的密码
:: www.XXXX.com 表示证书申请的域名
for /R %DIR% %%f in (*.node) do (
	%curdir%\HWInfo\signtool.exe sign /v /as /f %SignatureSecretKeyPath% /tr "http://sha256timestamp.ws.symantec.com/sha256/timestamp" /p password /fd sha256 /du https://www.XXXX.com /d . %%f
	if errorlevel 1 echo %errorlevel%
)

for /R %DIR% %%f in (*.dll) do (
	%curdir%\HWInfo\signtool.exe sign /v /as /f %SignatureSecretKeyPath% /tr "http://sha256timestamp.ws.symantec.com/sha256/timestamp" /p password /fd sha256 /du https://www.XXXX.com /d . %%f
	if errorlevel 1 echo %errorlevel%
)

for /R %DIR% %%f in (*.exe) do (
	%curdir%\HWInfo\signtool.exe sign /v /as /f %SignatureSecretKeyPath% /tr "http://sha256timestamp.ws.symantec.com/sha256/timestamp" /p password /fd sha256 /du https://www.XXXX.com /d . %%f
	if errorlevel 1 echo %errorlevel%
)

cd %curdir%
echo "done!"
goto Exit

:Error
echo Error!!!

:Exit
echo on



```

--- 


------------------------------------------------
**所有图片均来自网络**