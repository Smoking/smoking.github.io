---
title: Expression:(Invalid integer length modifier, 0)
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-27 13:37:27
img: https://i.loli.net/2019/06/27/5d1458cf5730097064.png
keywords: Expression,cpp,c++,vsnprintf,crash
summary:
categories: 编程
tags:
  - c++
  - windows
---

## 问题
最新公司项目桌面端软件上传一些特殊文件名的文件造成程序奔溃，找半天才找到奔溃的地方，竟然是vsnprintf这个函数奔溃了，知道奔溃的函数了那就debug一下，然后报错如下

![Expression:(Invalid integer length modifier, 0)](https://i.loli.net/2019/06/27/5d1458cf5730097064.png)

## 列上代码：

```cpp
void log::Write(LogLevel _level, const char* _tag, const char* _file, const char* _func, int _line, const char* _module,const char* format, ...)
{
    char buf[4096];
	va_list ap;
	va_start(ap, format);
	vsnprintf(buf, 4096, format, ap);
    writefile(TLogLevel(_level), _tag, _file, _func, _line, GetFormateLogHeader(format, buf).c_str());
    ...
}
```

## 原因
debug 之后发现 format 这个里面含有%20l 或者%d,就会出现问题，
那么是否就是说只要format中含有格式化输入的符号，后面的可变参数就自动变成输入参数了呢？
google搜索一圈发现 [vsnprintf() - MSDN - Microsoft](https://social.msdn.microsoft.com/Forums/vstudio/en-US/5326dae2-46aa-4290-9b9d-b456cdc636d4/vsnprintf?forum=vcgeneral) 也是这个问题

![vsnprintf crash](https://i.loli.net/2019/06/27/5d145c015b03663887.png)

**这说明问题不是出在vsnprintf 这个函数上，而是出在可变参数中的字符串中含有格式化输入的字符，只要使用可变参数的函数都会出现问题。**

## 解决方法

### 去掉可变参数
例如直接把函数改成
```
void log::Write(LogLevel _level, const char* _tag, const char* _file, const char* _func, int _line, const char* _module,const char* format)
```

但是我这底层调用还是可变参数，这个就不适合了，那就修改一下

```cpp
void log::Write(LogLevel _level, const char* _tag, const char* _file, const char* _func, int _line, const char* _module,const char* format, ...)
{
   xlogger2(TLogLevel(_level), _tag, _file, _func, _line,"%s", format);
    ...
}
```
用 %s这种方式可以处理了，但是有个弊端就是可变参数转发就不能实现了，而且只能把先把可变参数变成字符串

## 是否还有更好的方案呢？

有的话请告诉我吧。
