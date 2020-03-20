---
title: curl 自动获取并设置代理
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2020-03-20 09:27:35
img:
keywords: curl,c++
summary:
categories: 编程
tags:
  - C++
  - curl
---

### 自动从注册表中读取代理

```cpp
bool CCurlHttpClient::GetAutoProxy(vector<std::string> & proxyVct) {
  proxyVct.clear();
  HKEY key;
  auto ret = RegOpenKeyEx(HKEY_CURRENT_USER, R"(Software\Microsoft\Windows\CurrentVersion\Internet Settings)", 0, KEY_ALL_ACCESS, &key);
  if (ret != ERROR_SUCCESS) {
    //std::cout << "open failed: " << ret << std::endl;
    return false;
  }

  DWORD values_count, max_value_name_len, max_value_len;
  ret = RegQueryInfoKey(key, NULL, NULL, NULL, NULL, NULL, NULL,
    &values_count, &max_value_name_len, &max_value_len, NULL, NULL);
  if (ret != ERROR_SUCCESS) {
    //std::cout << "query failed" << std::endl;
    return false;
  }

  std::vector<std::tuple<std::shared_ptr<char>, DWORD, std::shared_ptr<BYTE>>> values;
  for (int i = 0; i < values_count; i++) {
    std::shared_ptr<char> value_name(new char[max_value_name_len + 1],
      std::default_delete<char[]>());
    DWORD value_name_len = max_value_name_len + 1;
    DWORD value_type, value_len;
    RegEnumValue(key, i, value_name.get(), &value_name_len, NULL, &value_type, NULL, &value_len);
    std::shared_ptr<BYTE> value(new BYTE[value_len],
      std::default_delete<BYTE[]>());
    value_name_len = max_value_name_len + 1;
    RegEnumValue(key, i, value_name.get(), &value_name_len, NULL, &value_type, value.get(), &value_len);
    values.push_back(std::make_tuple(value_name, value_type, value));
  }

  DWORD ProxyEnable = 0;
  for (auto x : values) {
    if (strcmp(std::get<0>(x).get(), "ProxyEnable") == 0) {
      ProxyEnable = *(DWORD*)(std::get<2>(x).get());
    }
  }

  if (ProxyEnable) {
    for (auto x : values) {
      if (strcmp(std::get<0>(x).get(), "ProxyServer") == 0) {
        std::cout << "ProxyServer: " << (char*)(std::get<2>(x).get()) << std::endl;
        std::vector<std::string> vctSub;
        std::string temp = (char*)(std::get<2>(x).get());
        CutString(temp, vctSub, ';');
        for (auto sub : vctSub)
        {
          if (sub.empty() == false && sub.find("https=")!= std::string::npos)
            proxyVct.push_back(replace_str(sub, "https=", ""));
        }
      }
    }
    if (proxyVct.size()>0)
      return true;
  }
  return false;
}
```

### 字符串截断

```cpp
void CCurlHttpClient::CutString(std::string line, std::vector<std::string> &subline, char a)
{
  //首字母为a，剔除首字母
  if (line.size() < 1)
  {
    return;
  }
  if (line[0] == a)
  {
    line.erase(0, 1);
  }

  std::size_t pos = 0;
  while (pos < line.length())
  {
    std::size_t curpos = pos;
    pos = line.find(a, curpos);
    if (pos == std::string::npos)
    {
      pos = line.length();
    }
    subline.push_back(line.substr(curpos, pos - curpos));
    pos++;
  }
  return;
}

```

### 字符串替换方法

```cpp
std::string CCurlHttpClient::replace_str(std::string str, const std::string before, const std::string after)
{
  for (std::string::size_type pos(0); pos != std::string::npos; pos += after.length())
  {
  pos = str.find(before, pos);
  if (pos != std::string::npos)
  return str.replace(pos, before.length(), after);
  else
  break;
  }
  return "";
}
```

### curl 设置代理方法

```cpp
//设置代理,自动读取代理
if (!GetAutoProxy(m_vctProxy))
  m_vctProxy.clear();
if (m_vctProxy.size()>0)
{
  for (auto strProxy:m_vctProxy)
  {
    curl_easy_setopt(easy_handle, CURLOPT_PROXY, strProxy.c_str());
  }
}
```
