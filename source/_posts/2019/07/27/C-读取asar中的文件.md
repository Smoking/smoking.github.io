---
title: C++ 读取asar中的文件
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-07-27 11:37:00
img: https://i.loli.net/2019/07/27/5d3bff01c6c5389215.jpg
keywords: asar,c++,node,js,electron,rapidjson,byte,uint32
summary:
categories: 编程
tags:
  - C++
  - nodejs
---

## 什么是asar

### 官方说明
>asar - Electron Archive
>Asar is a simple extensive archive format, it works like tar that concatenates all files together without compression, while having random access support.

>Features

>Support random access

>Use JSON to store files' information

>Very easy to write a parser

意思是说：asar类似于tar只是把所有文件合成一个文件而不进行压缩，所以理论上合成asar之后的总体积是要比原始文件要大的，毕竟还要加上存储文件信息
优点：
*   支持随机访问
*   使用JSON存储文件的信息
*   很容易编写解析器

**主要是electron中默认是使用这个来隐藏源码文件**
## asar文件结构

### 文件中的数据罗列

```
| UInt32: header_size | String: header | Bytes: file1 | ... | Bytes: file42 |
```

解析：
前8个字节表示信息头的长度，然后根据头的长度，去读文档中的结构json串，读取json串之后 按照文件中偏移位置和大小来读取指定文件的内容
### 文档信息结构json串

```json
{
   "files": {
      "tmp": {
         "files": {}
      },
      "usr" : {
         "files": {
           "bin": {
             "files": {
               "ls": {
                 "offset": "0",
                 "size": 100,
                 "executable": true
               },
               "cd": {
                 "offset": "100",
                 "size": 100,
                 "executable": true
               }
             }
           }
         }
      },
      "etc": {
         "files": {
           "hosts": {
             "offset": "200",
             "size": 32
           }
         }
      }
   }
}
```

## nodejs实现原理


```JavaScript
//读取asar文件头部信息源码
module.exports.readArchiveHeaderSync = function (archive) {
  const fd = fs.openSync(archive, 'r')
  let size
  let headerBuf
  try {
    const sizeBuf = Buffer.alloc(8)
    //先分配了8个字节读取头部需要的大小
    if (fs.readSync(fd, sizeBuf, 0, 8, null) !== 8) {
      throw new Error('Unable to read header size')
    }

    const sizePickle = pickle.createFromBuffer(sizeBuf)
    //注意:这里是转换成uint32的方式
    size = sizePickle.createIterator().readUInt32()
    headerBuf = Buffer.alloc(size)
    //读取header
    if (fs.readSync(fd, headerBuf, 0, size, null) !== size) {
      throw new Error('Unable to read header')
    }
  } finally {
    fs.closeSync(fd)
  }
  const headerPickle = pickle.createFromBuffer(headerBuf)
  //转换字符串
  const header = headerPickle.createIterator().readString()
  return { header: JSON.parse(header), headerSize: size }
}
```

在上面的代码有两个关键就是pickle这个模块的功能，已经怎么吧字节转换成headsize的,我们先看看 readUInt32的源码


```JavaScript
//pickle 中代码
var SIZE_INT32 = 4
var SIZE_UINT32 = 4
var SIZE_INT64 = 8
var SIZE_UINT64 = 8
var SIZE_FLOAT = 4
var SIZE_DOUBLE = 8
PickleIterator.prototype.readUInt32 = function () {
    return this.readBytes(SIZE_UINT32, Buffer.prototype.readUInt32LE)
  }
```
再看看Buffer.prototype.readUInt32LE 说明
>用指定的字节序格式（readUInt32BE() 返回大端序， readUInt32LE() 返回小端序）从 buf 中指定的 offset 读取一个无符号的 32 位整数值。

**重点在这里是小端序**

 * 大端序
   * 数据的高位字节存放在地址的低端 低位字节存放在地址高端
 * 小端序
   * 数据的高位字节存放在地址的高端 低位字节存放在地址低端
 什么意思：大端序是按照数字的书写顺序进行存储的，而小端序是颠倒书写顺序进行存储的。所以这里我们读取8位之后先倒叙，然后再转uint

再看两个源码
```javascript
PickleIterator.prototype.readString = function () {
    return this.readBytes(this.readInt()).toString()
  }

PickleIterator.prototype.readBytes = function (length, method) {
    var readPayloadOffset = this.getReadPayloadOffsetAndAdvance(length)
    if (method != null) {
      return method.call(this.payload, readPayloadOffset, length)
    } else {
      return this.payload.slice(readPayloadOffset, readPayloadOffset + length)
    }
  }

PickleIterator.prototype.getReadPayloadOffsetAndAdvance = function (length) {
    if (length > this.endIndex - this.readIndex) {
      this.readIndex = this.endIndex
      throw new Error('Failed to read data with length of ' + length)
    }
    var readPayloadOffset = this.payloadOffset + this.readIndex
    this.advance(length)
    return readPayloadOffset
  }
```
这段代码说明了什么，在头部这个串中是有位置偏移的，先读取要读的字符串长度，然后再读取制定的字符串。

## 解析流程

* 读取头8个字节
* 8个字节小端序转成 headsize
* 读取headsize 数量的二进制数据headbuf
* 读取headbuf前8个字节获取 head中json字符串的长度
* 解析json字符串的内容
* 根据json中文件信息读取制定位置的文件内容


## C++实现

### 读取header_size

```cpp
std::vector<byte> sizeBuf;
sizeBuf.resize(8);
FILE * file = nullptr;
_wfopen_s(&file,asarPath.c_str(), L"rb");
fread(&sizeBuf[0], sizeof(byte), 8, file);
//因为是小端序所有，先反转数组中的数据
std::reverse(sizeBuf.begin(), sizeBuf.end());
uint32_t t = deserialize1_uint32(&sizeBuf[4]);
```

### byte数组转uint32

```cpp
uint32_t deserialize1_uint32(unsigned char *buffer)
{
	uint32_t value = 0;
	value |= buffer[0] << 24;
	value |= buffer[1] << 16;
	value |= buffer[2] << 8;
	value |= buffer[3];
	return value;
}
```
### 根据headsize 读取head

```cpp
byte* headbuf = new byte[length];
memset(headbuf, 0, length);
rewind(file);
//从开始位置偏移8位开始读取
fseek(file,8,SEEK_SET);
fread(headbuf,sizeof(byte),length,file);
```

### 读取真实的json
```cpp
std::vector<byte> headSizeBuf;
headSizeBuf.resize(t);
memcpy(&headSizeBuf[0], &headbuf[4], 4);
//因为是小端序所有，先反转数组中的数据
std::reverse(headSizeBuf.begin(), headSizeBuf.end());
//获取真实的json串的长度
uint32_t readLength = deserialize1_uint32(&headSizeBuf[0]);
std::string headJson = std::string((char*)(&headbuf[8]), readLength);
```
### 解析json
C++解析json有很多的开源库，如rapidjson、jsoncpp等，这里我们使用rapidjson 方便引入，只需要引入头文件即可

### 遍历json 获取文件信息
```cpp
Document d;
d.Parse(headJson.c_str());
//第一个文件的偏移位置，是头8个字节加上head的长度
writeJson(file, length+8,u8"D:\\Test\\",d["files"]);
```

```cpp
void writeJson(FILE * file, long offset,std::string dir, rapidjson::Value &d)
{
	for (auto itr = d.MemberBegin();
		itr != d.MemberEnd(); ++itr)
	{
		std::cout << "-------------------当前目录:" << dir << "---------------------------------" << std::endl;
		createDirectory(dir);
		rapidjson::Value &value = itr->value;
		if (value.HasMember(u8"files") == true)
		{ //如果是文件夹就递归处理
			std::string strDir = dir + itr->name.GetString();
			strDir = strDir + u8"\\";
			writeJson(file, offset, strDir, value["files"]);
		}
		else {
			std::cout << "key:" << itr->name.GetString() << std::endl;
			size_t size, of;
			for (auto it = itr->value.MemberBegin();
				it != itr->value.MemberEnd(); ++it)
			{
                //获取offset 偏移量
				if (it->value.IsString()) {
					std::cout << "key:" << it->name.GetString() << "   value:" << it->value.GetString() << std::endl;
					of = atoi(it->value.GetString());
				}
                //获取size 文件大小
				else if (it->value.IsInt64()) {
					std::cout << "key:" << it->name.GetString() << "   value:" << it->value.GetInt64() << std::endl;
					size = it->value.GetInt64();
				}
			}
			std::vector<byte> sizeBuf;
			sizeBuf.resize(size);
			FILE * wfile = nullptr;
			std::wstring filepath = UTF8ToWide(dir + itr->name.GetString());
			_wfopen_s(&wfile, filepath.c_str(), L"wb");
			if (size != 0)
			{
                //从指定偏移位置读取数据
				fseek(file, offset + of, SEEK_SET);
                //读取制定的数据大小
				fread(&sizeBuf[0], sizeof(byte), size, file);
				fwrite(&sizeBuf[0], sizeof(byte), size, wfile);
			}
			fclose(wfile);
		}
	}
}
```


## 总结
asar还是很容易解析，但是也要注意一些细节，不然容易掉坑，为什么要用C++去解析asar文件，其实也是electron项目中，方便C++去做一些处理，如C++去运行asar中的js文件，C++去读取asar中的图片文件或者配置文件等。