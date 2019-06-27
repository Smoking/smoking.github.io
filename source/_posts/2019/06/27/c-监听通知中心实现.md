---
title: c++ 实现 事件监听通知中心
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-06-27 19:28:11
img: https://i.loli.net/2019/06/27/5d14b0f6472af16459.jpg
keywords: 事件监听,通知,事件注册,通知,监听,注册,libuv
summary:
categories: 编程
tags:
  - C++
  - windows 
---

## 目的
监听通知具体实现内容：当一个发送一个通知之后，所有监听这个事件的接口都收到通知执行某个任务
这里我使用libuv实现多线程执行任务，可以使用线程池替换实现，也可以使用单线程看具体使用场景

## 代码

```cpp
#include <map>
#include <mutex>
#include <vector>
#include <tuple>
#include <memory>
#include "uv.h"
class Notify
{
public:
    static Notify& Instance() { ///使用懒汉模式单利
        static Notify theSingleton;
        return theSingleton;
    }
public:
    //注册监听事件和函数，当有通知时会自动执行注册的函数
    //函数这里用智能指针防止注册的函数销毁了，没办法感知
    //obj指针为拥有注册函数的对象指针
    void Register(std::string strFuncName,void* obj,std::shared_ptr<void> spfunc)
    {
        std::lock_guard<std::mutex> lk(map_mutex_);
        auto it_find = func_map_.find(strFuncName);
        if (it_find == func_map_.end())
        {
            std::vector<std::tuple<void*, std::shared_ptr<void>>> vct;
            auto tp = std::tuple<void*, std::shared_ptr<void>>(obj, spfunc);
            vct.push_back(tp);
            func_map_.emplace(strFuncName, vct);
        }
        else {
            auto vct = func_map_[strFuncName];					
            auto tp = std::tuple<void*, std::shared_ptr<void>>(obj, spfunc);
            vct.push_back(tp);
            func_map_[strFuncName] = vct;
        }
        
    }
    //取消监听
    //obj指针为拥有注册函数的对象指针
    void UnRegister(void* obj,std::string strFuncName)
    {
        std::lock_guard<std::mutex> lk(map_mutex_);
        auto it_find = func_map_.find(strFuncName);
        if (it_find != func_map_.end())
        {
            auto vct = func_map_[strFuncName];
            
            for (auto it = vct.begin(); it!= vct.end();it++)
            {
                void* p = std::get<0>((*it));
                if (p == obj)
                {
                    vct.erase(it);
                    func_map_[strFuncName] = vct;
                    return;
                }
            }
        }
        
    }
    //执行通知函数，然后多线程去执行注册该事件的函数
    //这里用可变参数模板，因为这个要传递的参数个数是不确定的
    template <typename T, typename... Args>
    void ExcecuteNotify(std::string funcName, Args&&... args)
    {
        std::lock_guard<std::mutex> lk(map_mutex_);
        auto it_find = func_map_.find(funcName);
        if (it_find != func_map_.end())
        {
            auto vct = func_map_[funcName];
            
            for (auto it = vct.begin(); it != vct.end(); it++)
            {
                auto p = std::get<1>((*it));
                if (p!= nullptr)
                {
                    std::shared_ptr<T> sp = std::static_pointer_cast<T>(p);
                    auto pFunc = sp.get();
                    if (pFunc != nullptr)
                    {
                        //判断参数个数，跟进参数个数执行不同处理
                        if (sizeof...(args) > 0)
                        {
                            //
                            auto func = std::bind(*pFunc, std::forward<Args>(args)...);
                            uv_work_t * req = new uv_work_t();
                            req->data = &func;
                            uv_queue_work(uv_default_loop(), req, UvWorkCallBack, UvAfterWorkCallBack);
                        }
                        else {
                            uv_work_t * req = new uv_work_t();
                            req->data = pFunc;
                            uv_queue_work(uv_default_loop(), req, UvWorkCallBack, UvAfterWorkCallBack);
                        }
                        
                    }
                }
            }
            return;
        }
        
    }
    //释放所有注册函数
    void Release() {
        std::lock_guard<std::mutex> lk(map_mutex_);
        for (auto &it:func_map_)
        {
            it.second.clear();
        }
        func_map_.clear();
    }
    //执行相应的注册函数
    static void UvWorkCallBack(uv_work_t* req)
    {
        std::function<void()>* func = (std::function<void()>*)(req->data);
        (*func)();
    }
    //释放资源
    static void UvAfterWorkCallBack(uv_work_t* req,int status)
    {
        if (req != nullptr)
        {
            delete req;
            req = nullptr;
        }
    }
private:
    //用来保存注册的函数map
    std::map<std::string, std::vector<std::tuple<void*, std::shared_ptr<void>>>> func_map_;
    std::mutex map_mutex_;
private:
    Notify()                         // ctor hidden
    {
        Release();
    }
    Notify(Notify const&);            // copy ctor hidden
    Notify& operator=(Notify const&); // assign op. hidden
    ~Notify()
    {
        Release();
    }
};
```

## 调用

```cpp
//执行通知
using func = void(*)();
Notify::Instance().ExcecuteNotify<func>(GLOBAL_CLIENT_INFO_NOTIFY);

//注册函数
auto sp = std::make_shared<std::function<void(void)>>(std::bind(&classxx::classxx, this));
Notify::Instance().Register(GLOBAL_USER_INFO_NOTIFY, this, std::static_pointer_cast<void>(sp));
void classxx::classxx()
{

}
//也可以带参数
```

## 不足之处
注册函数时必须得知道通知时带的参数类型和个数，这个是弊端

有更好的办法请指点--

#### 如有错误欢迎指正,感激不尽

------------------------------------------------
**所有图片均来自网络**