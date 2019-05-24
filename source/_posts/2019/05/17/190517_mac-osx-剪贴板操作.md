---
title: mac osx 剪贴板操作
date: 2019-05-17 21:41:49
img: https://i.loli.net/2019/05/17/5cdebaef5dcf763513.jpg
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
keywords: macosx,objc,代码,编程,剪贴板
summary: macosx 实现剪贴板的打开、查询、赋值、清空的等基本操作
categories: macosx
tags:
  - objc
  - macosx
---

## 基本操作

### 获取剪贴板

```objectivec
NSPasteboard *pasteboard = [NSPasteboard generalPasteboard];
```

### 清除剪贴板

```objectivec
[pasteboard clearContents];
```

## 枚举剪贴板中的数据类型

```objectivec
NSPasteboard *pasteboard = [NSPasteboard generalPasteboard];
NSArray *urls = [[[NSPasteboard generalPasteboard] readObjectsForClasses:@[[NSURL class]] options:@{NSPasteboardURLReadingFileURLsOnlyKey: @(YES)}] valueForKey:@"path"];
if (urls.count > 0) {
    NSString *_nsStr = @"dropfile";
}
else  if ([pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypeRTF]] ||
          [pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypeRTFD]]){
    NSString *_nsStr = @"rtf";
}
else if ([pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypePNG]]||
         [pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypeTIFF]]) {
    NSString *_nsStr = @"image";
}
else if ([pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypeHTML]]) {
    NSString *_nsStr = @"html";
}
else if ([pasteboard canReadItemWithDataConformingToTypes:@[NSPasteboardTypeString]]){
    NSString *_nsStr = @"text";
}
else {
    NSString *_nsStr = @"orther";
}
```

## 从剪贴板中获取拷贝的文件地址

```objectivec
NSArray *urls = [[[NSPasteboard generalPasteboard] readObjectsForClasses:@[[NSURL class]] options:@{NSPasteboardURLReadingFileURLsOnlyKey: @(YES)}] valueForKey:@"path"];
for (NSInteger i= 0; i<urls.count; i++) {
    std::string _strTemp = [urls[i] UTF8String];
    NSLog(@"filepath:%s",_strTemp.c_str());
}
```

## 往剪贴板中写入数据

```objectivec
bool Clipboard::SetClipboardData(std::string _strFormatName, const char* _pBuffer, size_t _len)
{
    if(pasteboard == nil)
    {
        OpenClipboard();
    }
    NSData * _nsData = [NSData dataWithBytes:_pBuffer length:_len];
    if (_strFormatName == "text") {
        [pasteboard setData:_nsData forType:NSPasteboardTypeString];
    }
    else if(_strFormatName == "image") {
        [pasteboard setData:_nsData forType:NSPasteboardTypePNG];
    }
    else if(_strFormatName == "html") {
        [pasteboard setData:_nsData forType:NSPasteboardTypeHTML];
    }
    else if(_strFormatName == "rtf") {
        [pasteboard setData:_nsData forType:NSPasteboardTypeRTF];
    }
    else{
        NSString * _nsFormatName = [NSString stringWithCString:_strFormatName.c_str() encoding:NSUTF8StringEncoding];
        [pasteboard setData:_nsData forType:_nsFormatName];
        
    }
    return true;
}
```

## 往剪贴板中写入需要拷贝的文件地址

```objectivec
bool Clipboard::SetDropFileData(std::vector<std::string> _vctFile, bool _bCopyOrCut)
{
    pasteboard = [NSPasteboard pasteboardWithName:NSGeneralPboard];
    [pasteboard clearContents];
    NSMutableArray * _urlArry = [NSMutableArray array];
    for (auto it:_vctFile) {
        NSString *_nsstrTemp = [NSString stringWithUTF8String:it.c_str()];
        NSURL* _url = [NSURL fileURLWithPath:_nsstrTemp isDirectory:NO];
        [_urlArry addObject:_url];
    }
    [pasteboard writeObjects:_urlArry];
    return true;
}
```




[深圳利程电子有限公司](https://www.lcptcheater.com)


------------------------------------------------
**所有图片均来自网络**