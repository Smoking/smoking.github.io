---
title: innosetup MoveFile 失败 代码183
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-18 19:42:49
img: https://i.loli.net/2019/06/18/5d08ce60a6cdb11966.png
keywords: innosetup,MoveFile Failed,MoveFile失败,代码183
summary:
categories: 编程
tags:
  - 脚本
  - innosetup
---

![](https://i.loli.net/2019/06/18/5d08ce60a6cdb11966.png)

---

### 这类问题排查方案:

#### 权限对不对

安装包是否有权限去替换这个问题，如果没有权限提升权限验证试一下
innosetup 提升权限代码

```
    [Setup]
    PrivilegesRequired = admin
```

#### 安装包中是否存在这个文件

安装到其他路径对比查看是否有这个文件

#### 这个文件被其他软件占用

如果知道占用的程序，那么手动杀掉程序点击重试，是否能够成功。如果成功，那么只要代码去杀掉这个程序即可

##### 已知程序占用，则杀掉程序

innosetup 调用 KillTask函数 杀掉进程即可

##### 未知程序占用

innosetup 脚本代码先 调用rename 修改文件，再安装测试一下

```
if FileExists(tempfile)=false then
	  begin
        if RenameFile(oldfile,newfile) then
           begin
             result := true;
             Exit;
           end;
	  end;
```


