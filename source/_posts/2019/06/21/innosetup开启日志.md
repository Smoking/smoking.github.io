---
title: inno setup 开启 日志
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-21 10:38:01
img: https://i.imgur.com/BeFyqfn.png
keywords: innosetup,inno setup,log,日志,打开日志,开启日志,加入日志
summary:
categories: 编程
tags:
  - 脚本
  - innosetup
---


有时候安装程序的错误，我们无法跟踪需要日志功能，那么innosetup中如何开启日志呢

##### 步骤一
在setup模块中开启日志功能

```
[setup]
//打开日志功能
SetupLogging=yes
```

##### 步骤二

移动日志到指定位置方便查看

```
[setup]
procedure CurStepChanged(CurStep: TSetupStep);
var
  logfilepathname, logfilename, newfilepathname: string;
begin
  logfilepathname := ExpandConstant('{log}');
  logfilename := ExtractFileName(logfilepathname);
  newfilepathname := ExpandConstant('{app}\') + logfilename;

  if CurStep = ssDone then
  begin
    FileCopy(logfilepathname, newfilepathname, false);
  end;
end;

```

通过上述代码把日志放到安装目录下,如下图所示

![](https://i.loli.net/2019/06/21/5d0c492f54d4748726.png)


##### 部分日志内容如下

```
2019-06-21 11:00:11.923   Log opened. (Time zone: UTC+08:00)
2019-06-21 11:00:11.923   Setup version: Inno Setup version 5.5.1.ee2 (u)
2019-06-21 11:00:11.923   Original Setup EXE: C:\Users\28748\Downloads\CloudHubUpdate_1.1.9_1906211032.exe
2019-06-21 11:00:11.923   Setup command line: /SL5="$25138A,82682860,517632,C:\Users\28748\Downloads\CloudHubUpdate_1.1.9_1906211032.exe" /SPAWNWND=$440C0A /NOTIFYWND=$A513AC 
2019-06-21 11:00:11.923   Windows version: 6.2.9200  (NT platform: Yes)
2019-06-21 11:00:11.923   64-bit Windows: Yes
2019-06-21 11:00:11.923   Processor architecture: x64
2019-06-21 11:00:11.923   User privileges: Administrative
2019-06-21 11:00:11.925   64-bit install mode: No
2019-06-21 11:00:11.926   Created temporary directory: C:\Users\28748\AppData\Local\Temp\is-069S9.tmp
2019-06-21 11:00:15.382   Starting the installation process.
2019-06-21 11:00:15.556   Directory for uninstall files: D:\tmp\CloudHubX\阿萨法
2019-06-21 11:00:15.556   Will append to existing uninstall log: D:\tmp\CloudHubX\阿萨法\unins000.dat
2019-06-21 11:00:15.558   -- File entry --
2019-06-21 11:00:15.563   Dest filename: D:\tmp\CloudHubX\阿萨法\unins000.exe
2019-06-21 11:00:15.564   Time stamp of our file: 2019-06-21 11:00:11.401
2019-06-21 11:00:15.564   Dest file exists.
```
