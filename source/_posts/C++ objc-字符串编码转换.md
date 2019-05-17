---
title: C++ objc-字符串编码转换.md
date: 2019-05-17 21:27:17
author: smoking
img: https://i.loli.net/2019/05/17/5cdeb7f5dcc4d19912.jpg
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
summary: 在macosx 系统中C++字符串编码转换，利用objectc帮助转换实现
categories: macosx
tags:
  - C++
  - objc
  - macosx
---

最近碰到字符串编码转换的问题，简单记录下

### Ascii 转unicode

```objectivec
std::wstring AsciiToWide(std::string _strSrc)
{
    NSString *_nsstr = [NSString stringWithCString:_strSrc.c_str() encoding:NSASCIIStringEncoding];
    NSString *urlStringUTF8 = [_nsstr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    const char* _cdata = [urlStringUTF8 cStringUsingEncoding:NSUnicodeStringEncoding];
    std::wstring _wstr((wchar_t*)_cdata);
    return _wstr;
}
```



### unicode 转 Ascii

```objectivec
std::string  WideToAscii(std::wstring _strSrc)
{
    wchar_t* _csrc = const_cast<wchar_t*>(_strSrc.c_str());
    NSString *_nsstr = [NSString stringWithCString:(char*)_csrc encoding:NSUnicodeStringEncoding];
    NSString *urlStringUTF8 = [_nsstr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    const char* _cdata = [urlStringUTF8 cStringUsingEncoding:NSASCIIStringEncoding];
    std::string _str(_cdata);
    return _str;
}
```



### utf8 转 Ascii

```objectivec
std::string  UTF8ToAscii(std::string _strSrc)
{
    NSString *_nsstr = [NSString stringWithCString:_strSrc.c_str() encoding:NSUTF8StringEncoding];
    NSString *urlStringUTF8 = [_nsstr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    const char* _cdata = [urlStringUTF8 cStringUsingEncoding:NSUTF8StringEncoding];
    std::string _str(_cdata);
    return _str;
}
```



### Ascii 转 utf8

```objectivec
std::string  AsciiToUTF8(std::string _strSrc)
{
    NSString *_nsstr = [NSString stringWithCString:_strSrc.c_str() encoding:NSASCIIStringEncoding];
    NSString *urlStringUTF8 = [_nsstr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    const char* _cdata = [urlStringUTF8 cStringUsingEncoding:NSUTF8StringEncoding];
    std::string _str(_cdata);
    return _str;
}
```

[深圳利程电子有限公司](https://www.lcptcheater.com)
