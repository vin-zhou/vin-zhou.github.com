title: Xcode 编译设置
date: 2014-11-07 11:40:19
tags: Xcode
categories: 技术
---

Parameter |  说明
------ |-------
Link With Standard Libraries = YES |      用标准库连接
info.plist Output Encoding = binary | Info.plist输出编码为二进制
GCC_ENABLE_OBJC_GC = unsupported	| 垃圾回收关闭
Generate Debug Symbols = YES | 生成调试符号


编译器选择：
   Xcode 6.1 默认提供的是 Apple LLVM 6.0。 关于LLVM和GCC的恩怨，可见文章：[XCode 编译器介绍](http://www.cnblogs.com/ydhliphonedev/archive/2012/08/29/2661726.html)

## .pch文件（预编译头文件）
xcode项目中的每个文件都会自动导入这个预编译头文件，将不常改到的头文件放在此文件可以减少编译项目所需要的时间：
```objc
#ifdef __OBJC__
    #import <Foundation/Foundation.h>
#endif
```
## Info.plist文件
 属性列表文件，可以在这里修改程序的 名称、图标、版本、语言和程序支持的旋转方向等。
## InfoPlist.strings
用于多语言。

##iPhone 指令集

```
ARMv8/ARM64 = iPhone 5s, iPad Air, Retina iPad Mini
ARMv7s = iPhone 5, iPhone 5c, iPad 4
ARMv7  = iPhone 3GS, iPhone 4, iPhone 4S, iPod 3G/4G/5G, iPad, iPad 2, iPad 3, iPad Mini  
ARMv6  = iPhone, iPhone 3G, iPod 1G/2G
```
## 设置Xcode指令集
### Architectures
  指定工程将被编译成支持哪些指令集；支持越多，生成的指令集数据包越大。
### Valid Architecures
  指定可能支持的指令集，该列表和Architectures列表的交集，将是Xcode最终生成的二进制包所支持的指令集，比如
  Architectures支持了armv7s； Valid Architectures支持了： armv7/arm64/armv7s, 那么最终只支持armv7s。
### Build Active Architecture Only
  设置是否编译当前使用的设备对应的arm指令集。
  当为YES且你成功连上一个armv7指令集的设备时，此时将忽略Architectues和Valid Architectues的设置，只生成一个armv7指令集的二进制包。
  否则，仍根据Arch 和Valid Arch来设定。
  因此，通常情况下， 该选项在Debug模式下设为YES，便于使用调试设备；Release模式下设为NO.
## 如何选择支持的指令集
* 指令集都是可以向下兼容的；如支持armv7s、arm64指令集的机器上，可以跑低版本的armv7编译的程序。
* 如果你的软件对安装包大小非常敏感，可以减少支持的指令集数目；

##参考：

* [Xcode设置项之Architectures和Valid Architectures](http://wangzz.github.io/blog/2014/05/09/xcodeshe-zhi-xiang-zhi-architectureshe-valid-architectures/) ，这篇文章对Xcode的工程设置作为了较好的解释， 感谢原作者的辛勤工作！