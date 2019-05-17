---
title: mac osx 获取系统信息
date: 2019-05-17 21:34:10
img: https://i.loli.net/2019/05/17/5cdeb98cd001628055.jpg
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
summary: macosx 获取MAC地址、获取CPU序列号、获取操作系统序列号、获取硬盘序列号实现
categories: macosx
tags:
  - objc
  - macosx
---
#### 获取mac地址

```objectivec
bool ComputerDevice::GetMacAddress(std::vector<std::string>& vctMacAddress_)
{
	kern_return_t kr;
    CFMutableDictionaryRef matchDict;
    io_iterator_t iterator;
    io_registry_entry_t entry;

    matchDict = IOServiceMatching("IOEthernetInterface");
    kr = IOServiceGetMatchingServices(kIOMasterPortDefault, matchDict, &iterator);

    NSDictionary *resultInfo = nil;

    while ((entry = IOIteratorNext(iterator)) != 0)
    {
        CFMutableDictionaryRef properties=NULL;
        kr = IORegistryEntryCreateCFProperties(entry,
                                               &properties,
                                               kCFAllocatorDefault,
                                               kNilOptions);
        if (properties)
        {
            resultInfo = (NSDictionary *)properties;
            NSString *bsdName = [resultInfo objectForKey:@"BSD Name"];
            NSData *macData = [resultInfo objectForKey:@"IOMACAddress"];
            if (!macData)
            {
                continue;
            }

            NSMutableString *macAddress = [[NSMutableString alloc] init];
            const UInt8 *bytes = (const UInt8*)[macData bytes];
            for (NSUInteger i=0; i<macData.length; i++)
            {
                [macAddress appendFormat:@"%02x",*(bytes+i)];

            }

            //打印Mac地址
            if (bsdName && macAddress)
            {
                NSLog(@"网卡:%@\nMac地址:%@\n",bsdName,macAddress);
                vctMacAddress_.push_back([macAddress UTF8String]);
            }
        }
    }
    IOObjectRelease(iterator);
    return true;
}
```

#### 获取CPU 序列号

```objectivec
std::string ComputerDevice::GetCpuIdSerial()
{
    NSProcessInfo* pinfo = [NSProcessInfo processInfo];
    NSString* ret = [pinfo globallyUniqueString];
    return [ret UTF8String];
}
```

#### 获取操作系统序列号
```objectivec
std::string ComputerDevice::GetSoftSytemSerial()
{
	NSString * ret = nil;  
    io_service_t platformExpert ;  
    platformExpert = IOServiceGetMatchingService(kIOMasterPortDefault, IOServiceMatching("IOPlatformExpertDevice")) ;  
      
    if (platformExpert) {  
        CFTypeRef uuidNumberAsCFString ;  
        uuidNumberAsCFString = IORegistryEntryCreateCFProperty(platformExpert, CFSTR("IOPlatformSerialNumber"), kCFAllocatorDefault, 0) ;  
        if (uuidNumberAsCFString)   {  
            ret = [(NSString *)(CFStringRef)uuidNumberAsCFString copy];  
            CFRelease(uuidNumberAsCFString); uuidNumberAsCFString = NULL;  
        }  
        IOObjectRelease(platformExpert); platformExpert = 0;  
    }  
      
    std::string str = [ret UTF8String];
    [ret autorelease];
    return str;
}
```

#### 获取硬盘序列号


```objectivec

std::string ComputerDevice::GetHardWareSerial()
{
	
	NSString *ret = nil;  
    io_service_t platformExpert ;  
    platformExpert = IOServiceGetMatchingService(kIOMasterPortDefault, IOServiceMatching("IOPlatformExpertDevice")) ;  
      
    if (platformExpert) {  
        CFTypeRef serialNumberAsCFString ;  
        serialNumberAsCFString = IORegistryEntryCreateCFProperty(platformExpert, CFSTR("IOPlatformUUID"), kCFAllocatorDefault, 0) ;  
        if (serialNumberAsCFString) {  
            ret = [(NSString *)(CFStringRef)serialNumberAsCFString copy];  
            CFRelease(serialNumberAsCFString); serialNumberAsCFString = NULL;  
        }  
        IOObjectRelease(platformExpert); platformExpert = 0;  
    }  
    std::string str = [ret UTF8String];
    [ret autorelease];
    return str;
}
```



[深圳利程电子有限公司](https://www.lcptcheater.com)


