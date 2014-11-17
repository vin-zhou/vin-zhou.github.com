title: 探究Objective-C 之属性
tags: iOS
categories: 技术
date: 2014-11-09 22:26:46
---
在Objective-C 2.0，Apple提出了@property，一种成员变量属性标记语法，可以减少大量无谓的setter/getter代码。
## @property @synthesize
如可以对AllWeatherRadial的变量rainingHandling添加@property标记,
在.h文件中：
```objc
@inerface AllWeatherRadial:Tire
{
	float rainHandling;
}
@property float rainHandling;
@end // AllWeatherRadial.h
```
等效于在头文件中声明2个方法：
```objc
- (void) setRainHandling:(float)newRainHandling;
- (float)rainHandling;
```
在.m文件中：
```objc
#import "AllWeatherRadial.h"
@synthesize rainHandling;
```
等效于在实现文件中实现2个方法:
```objc
-（void）setRainHandling:(float)newRainHandling
{
 rainHandling = newRainHandling;
}
- (float) rainHandling
{
 return rainHandling;
} // AllWeatherRadial.m
```
标记``@property float rainHandling；``告诉编译器，可以通过-setRainHandling方法设置rainHandling变量，通过-rainHandling方法获得其变量值（相当于声明）。``@synthesize rainHandling``也是一个编译特性，让编译器添加-setRainHandling: 和-rainHandling的编译代码；而如果.m文件中已有了自定义的这两个方法，编译器就不会再次添加了（相当于实现）。

可以看到上例中包含@property的.h文件中，也声明了实例变量``{float rainHandling;}``,使用 @synthesize 设置的变量将会与之自动关联，并生成其setter/getter方法。可以在.h或.m文件中添加这些实例变量，区别在于：
* 如果在.h文件中添加，则其子类可以直接访问.h中的这些变量；（提供了公共访问接口）
* 如果在.m文件中添加，则只能本类访问。

如果.h或.m中没有声明这些变量，编译器会创建他们。因此可以把``{float rainHandling;}``从.h或.m文件中移除，只保留@property , @synthesize。

在Xcode4.5及以后的版本中，可以省略@synthesize这句，编译器会自动帮你加setter/getter方法，并且会默认访问_rainHandling这个成员变量，如果找不到，则会自动生成一个该名字的私有成员变量。但@synthesize可以定义与变量名不相同的set/get的命令,如``@synthesize rainHandling = anotherName``，在.m中以anotherName(而不是self.rainHandling)进行访问、赋值操作，保护变量不会被不恰当的访问。

调用 self.rainHandling = 1.0f;实质上调用的是rainHandling的setter函数，即``[self setRainHandling:1.0f];``。

## 在.m中给readonly的变量赋值
 例如：
 ```objc
@interface AllWeatherRadial:Tire
@property (readonly) NSString *title;  
@end // AllWeatherRadial.h
```
 编译器在看到readonly标记时，将会只声明getter函数而没有setter函数，因此在.m中无法直接给readonly的变量赋值。但可以通过以下2种方式实现：

* 方法1
   在.m中用@synthesize 声明一个同名的变量。
* 方法2
在.m中先声明一个变量如(mTitle)，再用@synthesize title = mTitle;绑定，由mTitle赋值。

## 使用@dynamic声明自定义setter/getter
如果想在运行时得到某个变量的值，而不需要编译器默认设置，可以使用@dynamic进行声明：
```objc
@property (readonly) float bodyMassIndex;

@dynamic bodyMassIndex;
- (float)bodyMassIndex
{
 // compute and return bodyMassIndex
}
```
如果使用了@dynamic声明，但没有提供accessor方法，将报错。
## 修改默认的accessor方法名
可以使用getter= 和 setter=来自定义方法名，如：
```objc
@property (getter = isHidden) BOOL hidden;
```
声明-(BOOL)isHidden；方法来取代默认的-(BOOL)hidden;

##@property语法
其语法为：``@property(参数1， 参数2)类型 名字；``
如：
```objc
@property(nonatomic, retain) UIWindow* window;
```
其中参数分为4类：
* 读写属性：readwrite (默认)/ readonly 
* setter语意： assign（默认/ retain / copy
* 原子性： atomicity(默认) / nonatomic

参数 | 意义 
------------ | ------------- 
readwrite | 产生setter/getter方法 
readonly | 只产生getter，没有setter
assign | setter方法直接赋值，而不进行retain操作
retain | setter方法对参数newValue执行  if（value != newValue){ [value release]; value = [newValue retain]}; 操作
copy | setter方法对参数newValue执行  if（value != newValue) { [value release];value = [newValue copy]; } 操作
nonatomic | 禁止多线程，变量保护，提高性能

* 使用assign: 对于基础数据类型（NSInteger)和C数据类型（int, float, double，char等）
* 使用copy: 对NSString
* 使用retain: 对其他NSObject和其子类

## strong / weak property
 strong和weak 属性只有在打开ARC时才会被要求使用，此时不能使用retain, release, autorelease；因此ARC会自动来做这些操作。但你需要在对象属性上使用strong和weak,其中
 * strong相当于retain(需要负责生命周期)，用了后， 引用计数自动+1，且不需要考虑是retain还是copy;
 * weak相当于assign（不需要），但它多了一点，会在内存释放后自动将变量设为nil。

默认情况下是strong。只有在避免retain cycle（循环引用）时，需要使用weak。

归属权问题：类内部的多用strong，外部的多用weak。

##其他
@property 只能用来简化设定accessor方法，对于多参数的如：
```objc
- (void)setTire:(Tire*)tire atIndex:(int)index;
- (Tire*)tireAtIndex:(int)index;
```
## 参考
* [Obejctive-C中的@property和@synthesize用法](http://justcoding.iteye.com/blog/1444548)
* [copy, retain等的使用](http://blog.csdn.net/duxinfeng2010/article/details/8461784)