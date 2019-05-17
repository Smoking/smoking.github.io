---
title: mac osx 涂抹马赛克实现
date: 2019-05-17 21:41:59
img: https://i.loli.net/2019/05/17/5cdebaef61d2896221.jpg
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
summary: macosx 涂抹位置变马赛克实现
categories: macosx
tags:
  - objc
  - macosx
---


## 实现原理
参考 [iOS手指涂抹位置变马赛克的实现](https://www.jianshu.com/p/e4bebae1b36f) 实现 mac osx 版本

## 图片马赛克实现
 借助CoreImage 库中的 CIFilter 滤镜，我们可以轻易实现图片的马赛克 代码如下

 ```objectivec

NSImage * img1 = [NSImage imageNamed:@"1.png"];
CGImageSourceRef source = CGImageSourceCreateWithData((__bridge  CFDataRef)img1.TIFFRepresentation, nil);
CGImageRef inImage = CGImageSourceCreateImageAtIndex(source, 0, nil);
CIImage *ciImage  = [[CIImage alloc] initWithCGImage:inImage];
CIFilter *filter  = [CIFilter filterWithName:@"CIPixellate"];
[filter setDefaults];
[filter setValue:ciImage forKey:kCIInputImageKey];
[filter setValue:@(10.f) forKey:kCIInputScaleKey];
CIImage * outPutImg = [filter outputImage];
NSCIImageRep * rep = [NSCIImageRep imageRepWithCIImage:outPutImg];
NSImage *img2 = [[NSImage alloc] initWithSize:[outPutImg extent].size];
[img2 addRepresentation:rep];

 ```

## CAShapeLayer 
通过图层遮罩来实现 根据Path 显示马赛克

```objectivec
//mac 使用layer 必须设置wantsLayer = YES，默认是NO
self.view.wantsLayer = YES;
//添加layer（imageLayer）到self上
self.imageLabyer = [CALayer layer];
self.imageLabyer.frame = self.view.bounds;
[self.view.layer addSublayer:self.imageLabyer];

self.shapeLayer = [CAShapeLayer layer];
self.shapeLayer.frame = self.view.bounds;
self.shapeLayer.lineCap = kCALineCapRound;
self.shapeLayer.lineJoin = kCALineJoinRound;
self.shapeLayer.lineWidth = 40.0f;
self.shapeLayer.strokeColor = [[NSColor blueColor] CGColor];
self.shapeLayer.fillColor = nil;
[self.view.layer addSublayer:self.shapeLayer];

//设置mask
self.imageLabyer.mask = self.shapeLayer;
self.path = CGPathCreateMutable();
//设置马赛克图片
self.imageLabyer.contents = img2;
//设置底图
self.view.layer.contents = img1;

```

## 鼠标响应处理
  设置View 监听鼠标移动消息
  `[self.view addTrackingRect:self.view.bounds
                    owner:self
                 userData:nil
             assumeInside:YES];`
 
 处理鼠标移动消息

 ```objectivec

- (void)mouseDown:(NSEvent *)event
{
    CGPoint pt = event.locationInWindow;
    CGPathMoveToPoint(self.path, NULL, pt.x, pt.y);
    CGMutablePathRef path = CGPathCreateMutableCopy(self.path);
    self.shapeLayer.path = path;
    CGPathRelease(path);
    NSLog(@"mouseDown");
}

-(void)mouseUp:(NSEvent *)event
{
    NSLog(@"mouseUp");
}
-(void)mouseDragged:(NSEvent *)event
{
    CGPoint pt = event.locationInWindow;
    CGPathAddLineToPoint(self.path, NULL,pt.x, pt.y);
    CGMutablePathRef path = CGPathCreateMutableCopy(self.path);
    self.shapeLayer.path = path;
    CGPathRelease(path);
    NSLog(@"mouseDragged");
}
    
  ```
## 效果图

![效果图](https://upload-images.jianshu.io/upload_images/1728667-c5e135087d60614b.gif?imageMogr2/auto-orient/strip)


[源码](https://github.com/Smoking/mosiac.git "源码")

[深圳利程电子有限公司](https://www.lcptcheater.com)