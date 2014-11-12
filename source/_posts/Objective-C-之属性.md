title: Objective-C 之属性
tags: iOS
categories: 技术
date: 2014-11-09 22:26:46
---
在Objective-C 2.0，Apple提出了@property，一种新的属性标记语法，可以减少大量的无谓代码。
如可以对AllWeatherRadial的属性rainingHandling添加@property标记：
```objc
@inerface AllWeatherRadial:Tire
{
	float rainHandling;
}
@property float rainHandling;
@end // AllWeatherRadial.h

---------
#import "AllWeatherRadial.h"
@synthesize rainHandling;

@end // AllWeatherRadial.m
```
替代set/get方法:
```objc
@inerface AllWeatherRadial:Tire
{
	float rainHandling;
}
- (void) setRainHandling:(float)rainHandling;
- (float)rainHandling;
@end // AllWeatherRadial.h
```
标记``@property float rainHandling；``就告诉编译器，可以通过-setRainHandling方法设置-rainHandling变量，通过rainHandling方法获得其变量值.``@synthesize rainHandling``也是一个编译特性，让编译器添加-setRainHandling: 和-rainHandling的编译代码；而如果.m文件中已有了自定义的这两个方法，编译器就不会再次添加了。

可以在.h和.m文件中添加property变量，区别在于：
* 如果在.h文件中添加，则
