---
title: 'git clone 之后,不做任何处理，却存在修改'
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2022-07-13 15:29:03
img:
keywords: git
summary:
categories: 编程
tags:
	- git
---

### 问题背景

换了个电脑，克隆一下我自己的一个仓库，clone 完成之后，发现本地有修改，但是我刚刚克隆的新仓库没做任何处理，怎么会有修改，很奇怪这个时候自然的去看看有什么修改

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   tags/C/index.html

no changes added to commit (use "git add" and/or "git commit -a")
```
这个时候看到上面clone 的时候有个警告
```shell
Cloning into 'smoking.github.io'...
remote: Enumerating objects: 1148, done.
remote: Counting objects: 100% (171/171), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 1148 (delta 100), reused 169 (delta 98), pack-reused 977
Receiving objects: 100% (1148/1148), 5.50 MiB | 187.00 KiB/s, done.
Resolving deltas: 100% (441/441), done.
warning: the following paths have collided (e.g. case-sensitive paths
on a case-insensitive filesystem) and only one from the same
colliding group is in the working tree:

  'tags/C/index.html'
  'tags/c/index.html'
```
关键信息在警告提示了，是路径发生冲突

### the following paths have collided (e.g. case-sensitive paths
on a case-insensitive filesystem)
	文件系统是不区分大小写的。所以，当你在同一目录下创建相同文件或文件夹会被告知文件已存在
	去gitlab上查看有修改的路径，发现确实是有相同文件名，但是是不同大小写的文件及文件夹。
	这样，问题就比较清晰了。
	那为什么会出现这种情况呢。因为这个项目是我和同事共同开发，冲突的那块是因为当时我先写了，他也用生成工具自动生成了。自动生成工具默认文件及文件名都是小写的，而我则采用驼峰之类的命名规范。 这就导致我们各自有一份，在合并到gitlab之后，gitlab并不认为这是两个相同的文件。所以，就保存两者了。然后，因为mac的文件系统是不区分大小写的，它会认为这是同一文件，所以就出现了上面的警告及修改。

### 解决方案

#### 第一种 修改文件名称然后提交仓库
#### 第二种 

+ 管理身份打开CMD
+ 进入到你需要保存工程代码的目录
+ 执行命令 fsutil.exe file SetCaseSensitiveInfo "C:\Users\Juan\Desktop" enable (这条命令表示为目录开启区分大小写)
+ 重新clone 你的代码
- (可选)删除或重命名冲突的文件和文件夹（如果它们相同）。您需要通过比较它们来验证这一点。要删除、使用git rm和移动或复制，请使用git mv
- (可选)提交并将您的更改推送到上游存储
- (可选)禁用区分大小写。命令：fsutil.exe file SetCaseSensitiveInfo "C:\Users\Juan\Desktop" disable
