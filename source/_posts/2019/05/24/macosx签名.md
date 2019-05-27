---
title: mac osx 数字签名
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
keywords: electron,mac,osx,签名,数字签名,代码签名,codesign
date: 2019-05-24 15:46:12
img: https://i.loli.net/2019/05/24/5ce7a8b7e828925086.jpg
summary:
categories: 编程
tags:
- macosx
- shell
---

---

## 签名证书获取

mac上文件签名需要获取 Developer ID Application 证书， 具体怎么获取证书参考[文章](<https://zhuanlan.zhihu.com/p/30844150>)  或者google 相关的文章即可 

**注意：和移动端IOS的证书不是一套**



## 证书导入

### 手动导入

​	双击证书，输入密码会自动导入，再钥匙访问串里面可以看到导入的证书

### 指令导入

```shell
# 将证书导入mac 系统钥匙串 再使用develoer id 来签名
#step1 解锁钥匙串
security unlock-keychain -p **** login.keychain

# 将证书文件导入到系统的login 钥匙串中 之后再使用原始的签名逻辑即可 P12Path p12证书文件路径 p12 注意证书不能有空格 需要将证书文件copy到此目录
security import "$SignatureSecretKeyPath" -k login.keychain -P **** -A
# **** 表示当前证书的密码
security unlock-keychain -p **** login.keychain
# **** 表示当前账号的密码
# 注意几个密码的区分
```



## 签名指令

```shell
codesign -s identity [-i identifier] [-r requirements] [-fv] [path ...]
codesign -v [-R requirement] [-v] [path|pid ...]
codesign -d [-v] [path|pid ...]
codesign -h [-v] [pid ...]
```



## 需要签名的内容

- mac 下的库文件 framework
- mac 下的动态库 dylib
- mac 下nodejs C++ 生成的 .node 文件
- 包含的 子app文件
- electron 本身带有的 库文件和子app 例如 Helper.app 等

## 脚本

```shell
#!/bin/bash -ilex
#脚本主要用来实现mac签名
#该函数递归查找 framework、dylib、node文件签名
function CodeSignFiles(){
    for element in `ls $1`
    do
        dir_or_file=$1"/"$element
        if [ -d $dir_or_file ]
        then
			strFileExName=${dir_or_file##*.}
			#echo "$strFileExName"
			#echo "$dir_or_file"
			if [[ "${dir_or_file##*.}"x = "framework"x ]];then
				echo ""
			else
				CodeSignFiles $dir_or_file
			fi;

        else
			strFileExName=${dir_or_file##*.}
			if [[ "${dir_or_file##*.}"x = "dylib"x ]];then
				#echo "$dir_or_file"
				codesign --force --verify --verbose --sign "$OSX_SIGN" "$dir_or_file"
			elif [[ "${dir_or_file##*.}"x = "node"x ]]; then
				#echo "$dir_or_file"
				codesign --force --verify --verbose --sign "$OSX_SIGN" "$dir_or_file"
			fi
        fi
    done
}

# 接收 electron-builder生成之后的 app 路径，以及 app 名字
App_path=$1
AppProductName=$2
if [ ! -n "$1" ] ;then
	echo "you should input app path...."
  	exit 1
fi

if [ ! -n "$2" ] ;then
	echo "you should input app name...."
  	exit 1
fi

#export OSX_SIGN = 'Developer ID Application: ****** Co., Ltd (*****)'
# 递归签名 electron-builder生成之后的 app 路径
CodeSignFiles "$App_path/Contents/Resources/"

# 签名electron app 自带的
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Mantle.framework/Versions/A"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/ReactiveCocoa.framework/Versions/A"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Squirrel.framework/Versions/A"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Electron Framework.framework/Versions/A"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Mantle.framework/"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/ReactiveCocoa.framework/"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Squirrel.framework/"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/Electron Framework.framework/"
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/$AppProductName Helper EH.app/" --timestamp=none
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/$AppProductName Helper NP.app/" --timestamp=none
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/Frameworks/$AppProductName Helper.app/" --timestamp=none
codesign --force --verify --verbose --sign "$OSX_SIGN" "$App_path/Contents/MacOS/$AppProductName" --timestamp=none

codesign -f -s "$OSX_SIGN" -v "$App_path" --deep --timestamp=none


# 查看签名状态 如果是 accep 则表示签名成功
echo "### verifying signature"
codesign -vvv -d "$App_path"
spctl -a -t exec -vv "$App_path"
```

---


------------------------------------------------
**所有图片均来自网络**