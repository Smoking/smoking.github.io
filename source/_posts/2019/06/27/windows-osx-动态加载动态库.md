---
title: windows、macosx 动态 加载 动态库 类封装
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-27 17:17:33
img: https://i.loli.net/2019/06/27/5d149696cc6a194563.jpg
keywords: 动态库,封装,完美转发,跨平台 加载 动态库,动态 加载 动态库,GetProcAddress,mac osx 加载动态库,windows 加载动态库,dlsym,模板类
summary:
categories: 编程
tags:
  - C++
  - windows 
---


## 目的
为了方便在windows平台或者mac、osx平台上调用动态库，封装一个C++基础模板类，实现动态库加载，函数获取以及直接调用，直接调用时只要传入函数名成和参数即可

因为要兼容windows 和mac 平台我们定义两个宏做编译区分
defined(OS_WIN)
defined(OS_MACOSX)

## 关键知识点
windows 加载动态库使用到的函数 LoadLibraryExW,GetProcAddress,FreeLibrary
macosx 加载动态库使用到的函数 dlopen,GetProcAddress,dlclose
可变参数模板函数
std::result_of<std::function<T>(Args...)>::type 
std::forward<Args>(args)...



## 代码

三个文件shared_library.h shared_library.cc shared_library.mm

### 头文件

```cpp
class  SharedLibrary
{
public:
    SharedLibrary();
    ~SharedLibrary();
public:
    //加载动态库dll或者dylib
    bool Load(std::string str_utf8_dllfilepath);
    //获取动态库中的函数指针
    //通过函数模板适应所以函数类型
    template <typename T>
    T* GetFunction(const std::string _funcName)
    {

        T* _funcPtr = nullptr;
#if defined(OS_WIN)
        //windows 中通过GetProcAddress 获取函数地址
        auto _ptr = GetProcAddress((HMODULE)libraryHandle_, _funcName.c_str());
        _funcPtr = (T*)_ptr;
#endif
#if defined(OS_MACOSX)
        //Mac dlsym 获取函数地址
        auto _ptr = (T*)dlsym(libraryHandle_, _funcName.c_str());
        _funcPtr = (T*)_ptr;
        const char *dlsym_error = dlerror();
        if (dlsym_error)
        {
            throw std::runtime_error(dlsym_error);
        }
#endif
        return _funcPtr;
    }
    //主动释放动态库
    void Free();
    //调用动态中的函数
    //这里使用可变参数模板，因为不同函数的参数不一样
    //std::result_of<std::function<T>(Args...)>::type 自动获取函数返回值类型
    template <typename T, typename... Args>
    typename std::result_of<std::function<T>(Args...)>::type ExcecuteFunc(const std::string funcName, Args&&... args)
    {
        //通过函数名称获取函数地址
        auto f = GetFunction<T>(funcName);
        if (f == nullptr)
        {
            std::string s = "can not find this function " + funcName;
        	throw std::runtime_error(s.c_str());
        }
        //使用完美转发，然后运行函数
        return f(std::forward<Args>(args)...);
    }		
private:
    //用于保存动态库加载之后的指针
    void* libraryHandle_;
    
};
```
### windows实现
```cpp
#include "shared_library.h"
#include <locale>
#include <codecvt>
#include <windows.h>
SharedLibrary::SharedLibrary()
{
    libraryHandle_ = nullptr;
}
SharedLibrary::~SharedLibrary()
{
    Free();
}
bool SharedLibrary::Load(std::string str_utf8_dllfilepath)
{
    std::wstring_convert<std::codecvt_utf8<wchar_t>> conv;
    std::wstring strpath = conv.from_bytes(str_utf8_dllfilepath);
    //转换成unicode编码加载路径，避免windows 平台 ascii路径存在的诸多问题
    //windows加载动态库使用LoadLibraryExW
    libraryHandle_ = LoadLibraryExW(strpath.c_str(), NULL, LOAD_WITH_ALTERED_SEARCH_PATH);
    if (libraryHandle_ == nullptr)
    {
        return false;
    }
    return true;
}

void SharedLibrary::Free()
{
    if (libraryHandle_ != nullptr)
    {
        ::FreeLibrary((HMODULE)libraryHandle_);
        libraryHandle_ = nullptr;
    }
}
```

### macosx 实现
```cpp
#include "shared_library.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dlfcn.h>
SharedLibrary::SharedLibrary()
{
    libraryHandle_ = nullptr;
}
SharedLibrary::~SharedLibrary()
{
    Free();
}
bool SharedLibrary::Load(std::string str_utf8_dllfilepath)
{
    //macosx 加载dylib使用的方式和linux一样
    libraryHandle_ = dlopen(str_utf8_dllfilepath.c_str(), RTLD_LOCAL|RTLD_LAZY);
    if (!libraryHandle_ )
    {
        printf("error msg:%s",dlerror());
        return false;
    }
    return true;
}

void SharedLibrary::Free()
{
    if (libraryHandle_ != nullptr)
    {
        //macosx 释放dylib使用的方式和linux一样
        dlclose(libraryHandle_);
        libraryHandle_ = nullptr;
    }
}
```


## 调用示例

```cpp
std::string dllfilepath = u8"";
#if defined(OS_WIN)  
    dllfilepath = _dllpath + u8"\\xxx.dll";
#elif defined(OS_MACOSX)  
    dllfilepath = _dllpath + u8"//xxx.dylib";
auto ptr_shared_library_ = new SharedLibrary();
if(ptr_shared_library_->Load(dllfilepath) == false)
{
    return false;
}

//传入函数名称和参数执行函数然后返回结果
auto _ptr = ptr_shared_library_->ExcecuteFunc<char*(int,int)>("func",1111,11111);
if (_ptr == nullptr)
{
    return false;
}
```

#### 如有错误欢迎指正,感激不尽

------------------------------------------------
**所有图片均来自网络**