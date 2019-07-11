---
title: inno setup 安装建立回滚机制
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-07-11 16:01:01
img: https://i.loli.net/2019/07/11/5d26f037048f774425.png
keywords:
summary:
categories: 编程
tags:
  - 脚本
  - innosetup
---

## 目的

安装包在安装过程中可能因为环境或者其他特殊原因安装失败了，但是不能影响以前安装的旧版本，所以就需要建立回滚机制，当安装失败的时候自动复原到以前的版本

## 实现步骤

* 在安装之前，拷贝安装目录下的文件到缓存目录，或者移动安装目录下的文件到缓存目录
* 安装完成之后判断，是否安装成功
* 如果安装成功
  * 删除缓存目录
* 如果安装失败
  * 把安装目录下的文件删除，从缓存目录拷贝文件到安装目录下
  * 删除缓存目录 


## 代码

### 定义全局变量

```
var
    isInstallSuccess:boolean; //用于判断是否安装成功
    tmpDir:String;       //缓存文件目录
```

### 移动目录

首先封装一个移动文件夹的函数
```
procedure DirectoryMove(SourcePath, DestPath: string);
var
  FindRec: TFindRec;
  SourceFilePath: string;
  DestFilePath: string;
begin
  if FindFirst(SourcePath + '\*', FindRec) then
  begin
    try
      repeat
        if (FindRec.Name <> '.') and (FindRec.Name <> '..') then
        begin
          SourceFilePath := SourcePath + '\' + FindRec.Name;
          DestFilePath := DestPath + '\' + FindRec.Name;
          if FindRec.Attributes and FILE_ATTRIBUTE_DIRECTORY = 0 then
          begin
            if RenameFile(SourceFilePath, DestFilePath) then
            begin
              Log(Format('Copied %s to %s', [SourceFilePath, DestFilePath]));
            end
              else
            begin
              Log(Format('Failed to copy %s to %s', [SourceFilePath, DestFilePath]));
            end;
          end
            else
          begin
            if DirExists(DestFilePath) or CreateDir(DestFilePath) then
            begin
              Log(Format('Created %s', [DestFilePath]));
              DirectoryMove(SourceFilePath, DestFilePath);
            end
              else
            begin
              Log(Format('Failed to create %s', [DestFilePath]));
            end;
          end;
        end;
      until not FindNext(FindRec);
    finally
      FindClose(FindRec);
    end;
  end
    else
  begin
    Log(Format('Failed to list %s', [SourcePath]));
  end;
end;
```

```

//拷贝现有工程目录下的文件到缓存目录
function copyAppFile(strInstallPath:String):Boolean;
begin
  tmpDir := ExpandConstant('{tmp}\CloudHubXSetupBackup');
  
  if DirExists(tmpDir)= true then  //如果文件夹存在那么先删除文件夹
    DelTree(tmpDir, False, True, True) //删除他下面所有的文件以及目录但是保存本身的目录
  else  //不存在那么先创建目录
    CreateDir(tmpDir);

  if DirExists(strInstallPath)= true then  //存在才拷贝
     DirectoryMove(strInstallPath,tmpDir);

end;
```
### 回滚文件

```
function rollBackFile(strInstallPath:String):Boolean;
begin

  if DirExists(tmpDir)= true then  //存在才移动
     DirectoryMove(tmpDir,strInstallPath);

end;
```

###  删除临时目录

```
function delRollBakcFileDir():Boolean;
begin
  if DirExists(tmpDir)= true then  //存在才删除
     DelTree(tmpDir, True, True, True);  
end;
```

### 判断

```
procedure CurStepChanged (CurStep: TSetupStep );
begin
    if(CurStep = ssDone ) then //最后安装程序退出之前
    begin
        if(isInstallSuccess = false) then  //如果异常退出那么还原文件
        begin
          MsgBox('{#MyAppNameZh}更新包异常退出', mbInformation, MB_OK);
          //回滚文件
          rollBackFile(oldinstallpath);
        end;
        //删除tmp文件夹内的文件
        delRollBakcFileDir();
    end
    else if(CurStep = ssPostInstall  ) then //实际安装完成之后
    begin
        isInstallSuccess := true;
    end
    else if(CurStep=ssInstall) then //开始执行安装之前
    begin
        //执行文件备份
        copyAppFile(oldinstallpath);
    end;
end;
```


## 总结
这里安装目录下没有对某些特殊的文件做屏蔽处理，根据需求可以做某些文件不放入回滚机制中，
使用移动文件的方式可以有效的减少innosetup安装过程中出现的文件被占用问题