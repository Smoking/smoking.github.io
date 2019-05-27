---
title: mac osx dock栏处理
date: 2019-05-17 21:33:55
img: https://i.loli.net/2019/05/17/5cdeb98cb476234625.jpg
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
keywords: macosx,objc,代码,编程,dock栏
summary: macosx 添加程序到dock栏，判断程序是否保留在dock栏，以及把程序从dock栏移出实现
categories: macosx
tags:
  - objc
  - macosx
---

#### 添加程序到dock栏


```objectivec
- (BOOL)addApplicationToDock:(NSString *)path {
    NSDictionary *domain = [self persistentDomainForName:@"com.apple.dock"];
    NSArray *apps = [domain objectForKey:@"persistent-apps"];
    NSArray *matchingApps = [apps filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"%K CONTAINS %@", @"tile-data.file-data._CFURLString", path]];
    if ([matchingApps count] == 0) {
        NSMutableDictionary *newDomain = [domain mutableCopy];
        NSMutableArray *newApps = [apps mutableCopy];
        NSDictionary *app = [NSDictionary dictionaryWithObject:[NSDictionary dictionaryWithObject:[NSDictionary dictionaryWithObjectsAndKeys:path, @"_CFURLString", [NSNumber numberWithInt:0], @"_CFURLStringType", nil] forKey:@"file-data"] forKey:@"tile-data"];
        [newApps addObject:app];
        [newDomain setObject:newApps forKey:@"persistent-apps"];
        [self setPersistentDomain:newDomain forName:@"com.apple.dock"];
        return [self synchronize];
    }
    return NO;
}
```


#### 判断程序是否保留在dock栏


```objectivec
- (BOOL)isApplicationOnDock:(NSString *)name{
    NSDictionary *domain = [self persistentDomainForName:@"com.apple.dock"];
    NSArray *apps = [domain objectForKey:@"persistent-apps"];
    NSArray *newApps = [apps filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"not %K CONTAINS %@", @"tile-data.file-data._CFURLString", name]];
    if (![apps isEqualToArray:newApps]) {
        return YES;
    }
    return NO;
}
```



#### 从dock栏移出


```objectivec
- (BOOL)removeApplicationFromDock:(NSString *)name {
    NSDictionary *domain = [self persistentDomainForName:@"com.apple.dock"];
    NSArray *apps = [domain objectForKey:@"persistent-apps"];
    NSArray *newApps = [apps filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"not %K CONTAINS %@", @"tile-data.file-data._CFURLString", name]];
    if (![apps isEqualToArray:newApps]) {
        NSMutableDictionary *newDomain = [domain mutableCopy];
        [newDomain setObject:newApps forKey:@"persistent-apps"];
        [self setPersistentDomain:newDomain forName:@"com.apple.dock"];
        return [self synchronize];
    }
    return NO;
}
```

[深圳利程电子有限公司](https://www.lcptcheater.com)

------------------------------------------------
**所有图片均来自网络**