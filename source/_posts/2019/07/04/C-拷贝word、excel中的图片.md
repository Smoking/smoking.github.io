---
title: C++ 获取 word、excel中拷贝的图片
author: Smoking
top: false
cover: false
coverImg: /medias/featureimages/3.jpg
toc: true
mathjax: false
date: 2019-07-04 15:21:01
img: https://i.loli.net/2019/07/04/5d1dadac85e9f89687.png
keywords: 
summary: 在word中拷贝图片，在剪贴板中存在的图片格式有几种情况，有CF_DIB,CF_BITMAP，CF_ENHMETAFILE，这里我们介绍获取CF_ENHMETAFILE 格式图片
categories:  编程
tags:
  - C++
  - windows 
---

## 目标

C++ 从剪贴板中获取在 word、excel中拷贝图片，然后再其他软件中显示该图片,一般bitmap,jpeg,png 是我们常用的渲染格式。所以获取的图片肯能需要转换成常用的格式才能显示

## 分析

在 word中拷贝图片，在剪贴板中可能存在几种格式如下

####  CF_DIB,CF_BITMAP  

在docx文件中拷贝一张图可能是这种格式

CF_DIB,CF_BITMAP 这类型的图片获取最简单

::GetClipboardData(format) 获取对应的bmp图片就可以了。



####  HTML Format 

在doc文件拷贝多张图是这种格式

我们看一个样例中的html 数据:

```html
<p class=MsoNormal ><img width="554"  height="268"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps26.png" ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><span><span style="position:absolute;z-index:1;margin-left:0.0000px;
margin-top:0.0000px;width:32.0000px;height:32.0000px;" ><img width="32"  height="32"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps27.png" ></span></span><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p></o:p></span></p><p class=MsoNormal ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><img width="555"  height="341"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps28.jpg" ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><img width="554"  height="310"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps29.jpg" ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><img width="554"  height="173"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps30.jpg" ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p><p class=MsoNormal ><img width="555"  height="305"  src="file:///C:\Users\28748\AppData\Local\Temp\ksohtml23516\wps31.jpg" ><span style="mso-spacerun:'yes';font-family:等线;font-size:10.5000pt;
mso-font-kerning:1.0000pt;" ><o:p>&nbsp;</o:p></span></p>
```

发现都是标签的形式，然后图片就是以本地文件路径放在标签src地址上，我们扫描信息获取标签可以获取图片地址，从而在其他软件中加载显示即可


####  CF_ENHMETAFILE  

在doc文件拷贝一张图可能是这种格式,Metafile 分为普通图元文件和增强型图元文件两种，扩展名分别为.wmf和.emf。图元文件将图形定义为编码的线段和图形，也称作“绘图类型”的图形。
我们看一下 metafile 的说明

> Metafile和位图的关系，就像点阵图和位元映射图形(矢量图形)的关系一样。点阵图通常来自实际的图像，而metafile则大多是通过电脑程式人为建立的。点阵是通过记录像素点的位置描绘图形，而矢量图形是通过数学公式即时演算画出的图形。Metafile由一系列与图形函式呼叫相同的二进位记录组成，这些记录一般用于绘制直线、曲线、填入的区域和文字等。

这篇文章目的就是获取这种格式的图片并且转换为其他格式

## 获取剪贴板中 CF_ENHMETAFILE 即 metafile 图片

直接上代码

```cpp

//打开剪贴板
::OpenClipboard(GetDesktopWindow());
if (::IsClipboardFormatAvailable(CF_ENHMETAFILE))
{
    //获取CF_ENHMETAFILE 格式图片
    HENHMETAFILE hemf = (HENHMETAFILE)::GetClipboardData(CF_ENHMETAFILE);
    HENHMETAFILE  hMetaFile=CopyEnhMetaFile(hemf, "c:\\test.emf");//保存到文件
    //关闭CMetafileDC并获得它的句柄
	DeleteEnhMetaFile(hMetaFile);
}
::CloseClipboard();

```

## 绘制 metafile 图片

```cpp

//打开剪贴板
::OpenClipboard(GetDesktopWindow());
if (::IsClipboardFormatAvailable(CF_ENHMETAFILE))
{
    //获取CF_ENHMETAFILE 格式图片
    HENHMETAFILE hemf = (HENHMETAFILE)::GetClipboardData(CF_ENHMETAFILE);
    HDC dc = ::GetDC(NULL);
    PlayEnhMetaFile(memDC, hemf, &rect); //绘制函数
}
::CloseClipboard();

```

## 读取 metafile 图片

```cpp

HENHMETAFILE hemf; 
hemf = GetEnhMetaFile("c:\\test.emf");
ENHMETAHEADER   emh; 
// Get the header from the enhanced metafile.   
ZeroMemory( &emh, sizeof(ENHMETAHEADER) );   
emh.nSize = sizeof(ENHMETAHEADER);   
if( GetEnhMetaFileHeader( hemf, sizeof( ENHMETAHEADER ), &emh ) == 0 )   
{   
    DeleteEnhMetaFile( hemf );   
    return FALSE;   
}
```

metafile 文件的头结构体如下 

```cpp

typedef struct tagENHMETAHEADER {
  DWORD iType;
  DWORD nSize;
  RECTL rclBounds;
  RECTL rclFrame;
  DWORD dSignature;
  DWORD nVersion;
  DWORD nBytes;
  DWORD nRecords;
  WORD  nHandles;
  WORD  sReserved;
  DWORD nDescription;
  DWORD offDescription;
  DWORD nPalEntries;
  SIZEL szlDevice;
  SIZEL szlMillimeters;
  DWORD cbPixelFormat;
  DWORD offPixelFormat;
  DWORD bOpenGL;
  SIZEL szlMicrometers;
} ENHMETAHEADER, *PENHMETAHEADER, *LPENHMETAHEADER;
```

具体每个元素的具体含义可以参考 [MSDN](https://docs.microsoft.com/en-us/windows/win32/api/wingdi/ns-wingdi-tagenhmetaheader) 

我们需要用到的是 rclFrame 这个参数获取图片的大小

## metafile 转 bitmap

```cpp
::OpenClipboard(GetDesktopWindow());
if (::IsClipboardFormatAvailable(CF_ENHMETAFILE))
{
    HENHMETAFILE hemf = (HENHMETAFILE)::GetClipboardData(CF_ENHMETAFILE);
    HBITMAP     bitmap;
    ENHMETAHEADER head;
    HDC         memDC;
    memset(&head, 0, sizeof(ENHMETAHEADER));
    head.nSize = sizeof(ENHMETAHEADER);

    GetEnhMetaFileHeader(hemf, sizeof(ENHMETAHEADER), &head);
    HDC dc = ::GetDC(NULL);
    RECT    rect;


    rect.left = 0;
    rect.top = 0;
    rect.right = head.rclFrame.right;
    rect.bottom = head.rclFrame.bottom;

    memDC = ::CreateCompatibleDC(dc);
    //Create a bitmap compatible to Window DC   
    bitmap = ::CreateCompatibleBitmap(dc, lWidth, lHeight);
 
    ::SelectObject(memDC, bitmap);
    
    //这里用白色的画刷，word中的透明图片会显示黑色

    SetBackColorToWhite(memDC);
    
    //把metafile 图片绘制到bitmap 上,绘制完成之后 bitmap就是我们需要的
    PlayEnhMetaFile(memDC, hemf, &rect);

}

```

## bitmap 转 png buffer
```cpp
bool save_png_memory(HBITMAP hbitmap, std::vector<BYTE> &data)
{
	Gdiplus::Bitmap bmp(hbitmap, nullptr);

	//write to IStream
	IStream* istream = nullptr;
	CreateStreamOnHGlobal(NULL, TRUE, &istream);

	CLSID clsid_png;
	CLSIDFromString(L"{557cf406-1a04-11d3-9a73-0000f81ef32e}", &clsid_png);
	Gdiplus::Status status = bmp.Save(istream, &clsid_png);
	if (status != Gdiplus::Status::Ok)
		return false;

	//get memory handle associated with istream
	HGLOBAL hg = NULL;
	GetHGlobalFromStream(istream, &hg);

	//copy IStream to buffer
	int bufsize = GlobalSize(hg);
	data.resize(bufsize);

	//lock & unlock memory
	LPVOID pimage = GlobalLock(hg);
	memcpy(&data[0], pimage, bufsize);
	GlobalUnlock(hg);

	istream->Release();
	return true;
}

```

调用代码

```cpp
std::vector<BYTE> data;
if (save_png_memory(bitmap, data))
{
    //write from memory to file for testing:
    std::ofstream fout("test.png", std::ios::binary);
    fout.write((char*)data.data(), data.size());
    fout.close();
}
```
## 总结

把上面的功能串起来之后 metafile 文件可以直接在内存中转化为bitmap，也可以转换成png、jpeg等需要的格式，不需要先保存到本地，减少io
