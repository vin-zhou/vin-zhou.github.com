title: 探究Objective-C 之深浅拷贝
tags: iOS
categories: 技术
date: 2014-11-17 15:03:24
---
## c中的浅拷贝与深拷贝
### 浅拷贝
  c中的浅拷贝就是对内存地址的复制，让目标对象指针和源对象指向同一片内存空间。如：
```c
char *str = (char*)malloc(100);
char* str2 = str;
```
 浅拷贝只是对对象的简单拷贝，让几个对象共用同一片内存，当内存销毁时，指向这片内存的几个指针需要重新定义才可以使用，要不然成为野指针。

### 深拷贝
 c中的深拷贝是指拷贝对象的内容也重新分配一块地址，两个对象也互不影响、互不干涉。如：
```c
char *str = (char*)malloc(100);
char *str2 = (char*)malloc(100);
memcpy(str2, str, sizeof(char)* 100);
``` 
##iOS中的浅拷贝与深拷贝
###集合类对象

对于如NSArray等的集合对象，iOS官方的深浅拷贝方法交代的非常清楚：https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Collections/Articles/Copying.html

总结起来如下：
#### 浅拷贝
```objc
NSArray *shallowCopyArray = [someArray copyWithZone:nil];
// or
NSDictionary *shallowCopyDict = [[NSDictioary alloc] initWitheDictionary:someDictioanry copyItems: NO];
```
#### 深拷贝
```objc
NSArray *deepCopyArray = [NSArray alloc] initWitheArray: someArray copyItems: YES]; // 只深拷贝了一层，如果数组的元素也为 集合类型， 则无法完全拷贝
// or
NSArray *trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject: oldArray]]; // 完全拷贝
```
###NSString、NSMutableString对象
NSString对象的retain非常特殊。具体可参考：[Objective-C中NSString对象retainCount之谜探索](http://blog.csdn.net/developer_zhang/article/details/9342647)
总结起来是：

* @式
  ```objc
  NSString *s = @"t";
  NSLog(@"s:%lx",[s retainCount]);//输出值为0xffffffffffffffff(UINT_MAX, 2147483647)
  NSLog(@"s:%ld",[s retainCount]);//输出值-1,由于0xffffffffffffffff补码表示的值为-1
  ```
  s指向的是字符串常量（类似于c语言中的静态区存储），系统不做引用计数，任何retain\release操作，其retainCount值都为UINT_MAX。
* stringWithFormat
 ```objc
 NSString *s = [NSString stringWithFormat:@"%s", "test"];
 NSLog(@"s:%d",[s retainCount]); //输出值为1
 ```
 系统会正常使用引用计数，和普通对象一样；
* stringWithString
 取决于后面的string对象；
 1）
 ```objc
 NSString *s1 = [NSString stringWithString:@"test"];
 NSLog(@"s1:%d",[s1 retainCount]); // 2147483647
 ```
 如“test"为常量，则s1也指向常量，retain为UINT_MAX;
 2)
 ```objc
 NSString *s2 = [NSString stringWithString:[NSString stringWithFormat:@"test,%d",1]];
 NSLog(@"s2:%d",[s2 retainCount]); // 2
 如先用stringWithFormat生成了一个变量，retain为1，再用stringWithString，retain增为2.
 ```
* NSMutableString
 ```objc
 NSMutableString* myStr3 = [NSMutableString stringWithString:@"string 3"]; // 1
 ```
 使用stringWithString，正常计数。

因此 这里只对使用stringWithFormat式创建的对象的copy/mutblecopy行为进行研究。
```objc
    T* source = [T stringWithFormat: @"test"];
    T* dest = [source copy/mutablecopy];
```
1. 使用copy，无论source\dest对象是否为mutable，都会退化为immutable；且执行的是浅拷贝，指向同一个老的对象， retain数+1；
2. 使用mutablecopy，只有在source和dest都为NSMutableString时，可正常使用；且执行的是深拷贝，生成新对象，retain数为1。

## iOS其他对象的深浅拷贝

### NSCopying 
 iOS中的immutable对象实现的是NSCopying协议，苹果官网：[NSCopying](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSCopying_Protocol/index.html#//apple_ref/doc/uid/TP40003777) 讲的非常清楚，
在实现该协议的copyWithZone方法时， 不同情况下有不同的实现方式，导致的retain数变化也不定相同：
1. 若没有实现过copyWithZone, 则可直接使用alloc和init创建新对象；
2. 若该类继承的父类实现了copyWithZone方法，则需overwrite该方法，先调用父类的该方法，再对新增的变量赋值；
3. 若为immutable对象，则直接retain+1,返回原始对象；（如同NSString)
例如：

```objc
#import <Foundation/Foundation.h>

@interface Tire : NSObject<NSCopying>

@property float pressure;
@property float treadDepth;

// ... methods;
- (void) initWithPressure:(float)pressure treadDepth:(float) treadDepth;
- (id)copyWithZone:(NSZone *)zone;

@end
----------------
#import "Tire.h"

@implementation TestObj

-(id)copyWithZone:(NSZone *)zone
{
    Tire *copy = [[[self class] allocWithZone:zone]initWithPressure:pressure
     treadDepth: treadDepth];

    return copy;
}
@end
```
这里使用了类方法 ``+ (id) allocWithZone: (NSZone *) zone;``，这个消息将被发送到类，而不是实例。 这里使用了``[self class]``而不是``Tire``，是因为拷贝时，如果是给其子类发送消息，则将无法拷贝其子类的完整信息。
这里调用了``initWithPressure:treadDepth:``方法，也可以先调用``init``，再手动赋值：
```objc
 copy.pressure = pressure;
 copy.treadDepth = readDepth;
```
**对于子类,其实现Copying的例子如下：**
```objc
@interface AllWeatherRadial : Tire
// ... propertie
@property float rainHandling;
@property float snowHandling;
// ... methods
@end // AllWeatherRadial
-----------------------
-(id)copyWithZone: (NSZone *) zone
{
	AllWeatherRadial *tireCopy;
    tireCopy = [super copyWithZone : zone];
    tireCopy.rainHandling = rainHandling;
    tireCopy.snowHandling = snowHandling;
    return tireCopy;
}
```
因为AllWeatherRadial继承自一个已实现NSCopying的父类，因此.h里不必再做``<NSCopying>``协议声明，而且.m中也没有使用之前的[[self class]allocWithZone]方法，而是使用了父类的copyWithZone方法，由于该方法返回的[self class]类型，因此符合AllWeatherRadial。之后，手动给新添加的属性赋值。

### NSMutableCopying
 对于mutable对象，实现的是NSMutableCopying协议，苹果官网[NSMutableCopying](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSMutableCopying_Protocol/index.html#//apple_ref/doc/uid/TP40003783)。

如果该类的父类实现了mutableCopyWithZone：方法， 则需要重写该方法，先调用父类的该方法，再对新增的变量赋值；



## 参考
* [Objective-C中的浅拷贝和深拷贝](http://www.cocoachina.com/ios/20141113/10213.html), 其大部分内容有误
* [Objective-C中NSString对象retainCount之谜探索](http://blog.csdn.net/developer_zhang/article/details/9342647)