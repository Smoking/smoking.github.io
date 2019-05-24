---
title: electron代码解密
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-05-24 15:45:51
keywords: electron,代码,加密,解密,nodejs,c++,openssl,AES
img: https://i.loli.net/2019/05/24/5ce7a7bfbbe6458325.jpeg
summary:
categories: 编程
tags:
- C++
- electron
---


## 解密模块需要解决的问题

- 在读取指定路径的加密文件
- AES 解密代码文件
- 加载 JS 解密之后的内容

### 怎么加载

[nodejs模块加载机制](https://segmentfault.com/a/1190000012373889)

[require() 源码解读](http://www.ruanyifeng.com/blog/2015/05/require.html)

通过了解nodejs 的模块加载机制 其实执行的主要代码类似

```javascript
(function (exports, require, module, __filename, __dirname) {
  // 模块源码
});
```

那么，我们在C++中也用此模型来加载代码文件

## nodejs C++ 插件开发

这一块的学习资料具体参考[官方文档](<http://nodejs.cn/api/addons.html>)和[官方例子](https://github.com/nodejs/node-addon-examples.git)

nodejs C++插件中 我们是可以使用V8的，这是我们执行 js 代码的重要前提条件



## 获取加密JS代码路径

```cpp
//file_path 需要返回的完整路径
//resources_dir_path 运行时获取的electron资源文件夹路径
//加密之后的js文件md5值

bool LoadJS::GenerateFilePath(std::string& file_path, const std::string& resources_dir_path, const std::string& code_id)
{
    if (resources_dir_path.length() > 0 && code_id.length() > 0)
    {
        file_path = resources_dir_path;
        //加密的代码不能放入asar 中，不然C++无法加载，这里的路径为加密文件时定义好的路径
        file_path += u8"/app.asar.unpacked/encryption/";
        file_path += code_id;
        return true;
    }
    return false;
}
```

## AES 解密实现

通过 openssl来实现AES 解密，下面这个例子，来自[openssl官方文档](https://www.openssl.org/docs/manmaster/man3/EVP_CipherUpdate.html])。

```c
int do_crypt(FILE *in, FILE *out, int do_encrypt)
 {
     /* Allow enough space in output buffer for additional block */
     unsigned char inbuf[1024], outbuf[1024 + EVP_MAX_BLOCK_LENGTH];
     int inlen, outlen;
     EVP_CIPHER_CTX *ctx;
     /*
      * Bogus key and IV: we'd normally set these from
      * another source.
      */
     unsigned char key[] = "0123456789abcdeF";
     unsigned char iv[] = "1234567887654321";

     /* Don't set key or IV right away; we want to check lengths */
     ctx = EVP_CIPHER_CTX_new();
     EVP_CipherInit_ex(&ctx, EVP_aes_128_cbc(), NULL, NULL, NULL,
                       do_encrypt);
     OPENSSL_assert(EVP_CIPHER_CTX_key_length(ctx) == 16);
     OPENSSL_assert(EVP_CIPHER_CTX_iv_length(ctx) == 16);

     /* Now we can set key and IV */
     EVP_CipherInit_ex(ctx, NULL, NULL, key, iv, do_encrypt);

     for (;;) {
         inlen = fread(inbuf, 1, 1024, in);
         if (inlen <= 0)
             break;
         if (!EVP_CipherUpdate(ctx, outbuf, &outlen, inbuf, inlen)) {
             /* Error */
             EVP_CIPHER_CTX_free(ctx);
             return 0;
         }
         fwrite(outbuf, 1, outlen, out);
     }
     if (!EVP_CipherFinal_ex(ctx, outbuf, &outlen)) {
         /* Error */
         EVP_CIPHER_CTX_free(ctx);
         return 0;
     }
     fwrite(outbuf, 1, outlen, out);

     EVP_CIPHER_CTX_free(ctx);
     return 1;
 }
```



## 加载加密的代码内容

```cpp
//获取解密的key 和向量 iv
bool LoadJS::GenerateKeyAndIv(unsigned char* key, unsigned char*& iv, const std::string& code_id)
{
    if (code_id.length() == 32 && key != nullptr)
    {
        const char* ch_code_id = code_id.c_str();
        key = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        iv = (unsigned char*)code_id.c_str();
        iv += 4;
        return true;
    }
    return false;
}



//file_data 文件内容
//file_length 文件大小
//need_decrypt 是否需要解密
//file_path 加密文件路径
// code_id 加密文件md5 值
bool LoadJS::LoadFileData(char*& file_data, int& file_length, const bool& need_decrypt, const std::string& file_path, const std::string& code_id)
{
    bool success = false;
    FOpen file_open;
    //打开文件
    file_open.Open(file_path, FOpen::R_B_MODE_TYPE);
    if (file_open.IsOpen())
    {
        unsigned int file_decrypt_length = file_open.GetLength();
        if (file_decrypt_length <= 0)
        {
            return false;
        }
        //设置缓存大小
        char* ch_file_decrypt_data = new char[(unsigned int)file_decrypt_length];
        //读取加密文件内容
        file_open.ReadChar(ch_file_decrypt_data, file_decrypt_length);
        file_open.Close();

        if (need_decrypt)
        {
            //
            unsigned char* ch_key = new unsigned char[33];
            ch_key[32] = '\0';
            unsigned char* ch_iv = nullptr;
            if (GenerateKeyAndIv(ch_key, ch_iv, code_id))
            {
                //解密缓存ch_file_decrypt_data 中的内容
                //把openssl 官方加解密的例子稍微封装一下
                if (Decrypt((const unsigned char*)ch_file_decrypt_data, file_decrypt_length, (unsigned char*&)file_data, (unsigned int&)file_length, ch_key, ch_iv, false))
                {
                    success = true;
                }
            }
            if (ch_key != nullptr)
            {
                memset(ch_key, 0, sizeof(unsigned char) * 32);
                delete[] ch_key;
            }

            if (ch_file_decrypt_data != nullptr)
            {
                memset(ch_file_decrypt_data, 0, sizeof(char) * file_decrypt_length);
                delete[] ch_file_decrypt_data;
            }
        }
        else
        {
            file_data = ch_file_decrypt_data;
            file_length = file_decrypt_length;
            success = true;
        }
    }
    return success;
}
```

## 加载 JavaScript 

```cpp
void LoadJS::V8Load(const Nan::FunctionCallbackInfo<v8::Value>& info)
{
    bool success = false;			 // 前置返回值
    //v8 对象域
    Nan::HandleScope scope;
    if (info.Length() < 9)
    {
        Nan::ThrowError("Wrong number of arguments");
        return;
    }
    //获取Global 对象 
    v8::Local<v8::Object> global = Nan::GetCurrentContext()->Global();
    //获取传入的参数
    v8::Local<v8::Value> need_decrypt = info[0];
    v8::Local<v8::Value> resources_dir_path_arg = info[1];
    v8::Local<v8::Value> code_id_arg = info[2];
    v8::Local<v8::Value> exports_arg = info[3];
    v8::Local<v8::Value> require_arg = info[4];
    v8::Local<v8::Value> module_arg = info[5];
    v8::Local<v8::Value> file_name_arg = info[6];
    v8::Local<v8::Value> dir_name_arg = info[7];
    v8::Local<v8::Value> process_arg = info[8];
    //参数判断你是否 传入正确
    if (!need_decrypt->IsBoolean() || !resources_dir_path_arg->IsString() || !code_id_arg->IsString() || !require_arg->IsFunction() || !file_name_arg->IsString() || !dir_name_arg->IsString())
    {
        Nan::ThrowError("Wrong argument");
        return;
    }
	//判断md5值是否为32 位
    v8::Local<v8::String> v8_str_code_id = code_id_arg.As<v8::String>();
    if (v8_str_code_id->Length() != 32)
    {
        Nan::ThrowError("Wrong code_id length");
        return;
    }

    // 保存运行时electron的资源文件夹路径 变量
    std::string str_resources_dir_path;
    if (v8convert::v8String2UTF8String(str_resources_dir_path, resources_dir_path_arg.As<v8::String>()))
    {
        // 保存 加密文件md5值 变量
        std::string str_code_id;
        if (v8convert::v8String2UTF8String(str_code_id, code_id_arg))
        {
            std::string str_error;
            std::string str_file_path;
            if (GenerateFilePath(str_file_path, str_resources_dir_path, str_code_id))
            {
                char* ch_file_data = nullptr;
                int file_length = 0;
                bool b_need_decrypt = need_decrypt->BooleanValue();
                //代码文件解密
                if (LoadFileData(ch_file_data, file_length, b_need_decrypt, str_file_path, str_code_id))
                {
                    std::string str_code = "(function (global, exports, require, module, __filename, __dirname, process) { return function (global, exports, require, module, __filename, __dirname, process) {\n";
                    str_code += std::string(ch_file_data, file_length);
                    str_code += "\n}.call(this, global, exports, require, module, __filename, __dirname, process); })";
                    memset(ch_file_data, 0, sizeof(char) * file_length);
                    if (b_need_decrypt)
                    {
                        free(ch_file_data);
                    }
                    else
                    {
                        delete[] ch_file_data;
                    }
                    v8::Local<v8::String> l_v8_str_code = Nan::New(str_code).ToLocalChecked();
                    memset((void*)str_code.c_str(), 0, sizeof(char) * str_code.size());
                    v8::Local<v8::Script> l_script_code;
                    //C++ 字符串 JavaScript脚本转换成v8执行脚本
                    v8::MaybeLocal<v8::Script> ml_script_code = Nan::CompileScript(l_v8_str_code);
                    if (ml_script_code.ToLocal(&l_script_code))
                    {
                        if (!l_script_code.IsEmpty())
                        {
                            // 执行获得函数
                            v8::Local<v8::Value> l_v8_script_code_run_result = l_script_code->Run();
                            if (l_v8_script_code_run_result->IsFunction())
                            {
                                v8::Local<v8::Function> l_v8_script_code_fun = l_v8_script_code_run_result.As<v8::Function>();
                                //参数列表
                                v8::Local<v8::Value> l_v8_code_argv[] = { global, exports_arg, require_arg, module_arg, file_name_arg, dir_name_arg, process_arg };
                                // v8 调用函数
                                l_v8_script_code_fun->Call(global, 7, l_v8_code_argv);
                                success = true;
                            }
                            else
                            {
                                str_error = u8"script_code is not function:";
                            }
                        }
                        else
                        {
                            str_error = u8"script_code is empty:";
                        }
                    }
                    else
                    {
                        str_error = u8"compileScript failed:";
                    }
                }
                else
                {
                    str_error = u8"loadFileData failed:";
                }
            }
            else
            {
                str_error = u8"generateFilePath failed:";
            }
            if (!success)
            {
                str_error += u8" codeid = ";
                str_error += str_code_id;
                Nan::ThrowError(str_error.c_str());
            }
        }
    }
    info.GetReturnValue().Set(Nan::New(success));
}
```

## 执行说明

上面C++ 代码其实执行的是这样一段JS 代码

```javascript
(function (global, exports, require, module, __filename, __dirname, process) 
{ 
    return function (global, exports, require, module, __filename, __dirname, process) 	   {
		// 解密后的JS代码内容
    }.call(this, global, exports, require, module, __filename, __dirname, process); 
}
)
```

然后执 闭包，最后调用函数

## 总结

把加密封装成一个nodejs模块，然后生产环境打包时完成代码加密，运行时依赖它解密即可，加密和解密配套使用，还可以使用zip等压缩包配套再来一层加密

至此，electron的源码加解密全部完成

---


------------------------------------------------
**所有图片均来自网络**