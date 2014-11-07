title: Objective-C-之对象初始化
date: 2014-11-06 21:54:36
tags: iOS
categories: 技术
---
## 分配对象空间
Allocation: 给一个类发送alloc消息将会导致该类分配一块足够大的内存空间来存放其所有实例；同时内存会被初始化设为0.即：BOOL被设为NO； int 设为0；float 设为 0.0；指针设为 nil.

一个被alloc的对象并不能马上使用，还需要调用init方法。在C++、Java等语言中，对象的内存分配和初始化都被在一个构造函数中实现；Objective-C将其分为了2步。如果只使用alloc，没有init，则很可能出现奇怪的错误。你应该在alloc后，链式使用init函数如：
```objc
  Car* car = [[Car alloc] init];
  [Car new]; // 这种方式也可以，但严重不推荐！
```
而不是
```objc
  Car* car = [car alloc]; [car init];
```
这是因为 init得到的对象可能和alloc得到的并不是同一个。在有些情况， 如 NSString 的init方法会根据不同的参数（非常长的字符串亦或是阿拉伯字母）来选择要构造的对象，并替代原来的对象。
##init 方法
例如：
```objc
 (id) init
 {
 	if(self = [super init])
    {
    	engine = [[Engine alloc] init];
        tries[0] = [[Tire alloc] init];
        tries[1] = [[Tire alloc] init];
        tries[2] = [[Tire alloc] init];
        tries[3] = [[Tire alloc] init];
    }
    return (self);
 } // init
```
 使用 ``self = [super init]`` 是因为先要保证父类的已被初始化，且 “Instance variables are found at a memory location that's a fixed distance from the hidden self parameter. If a new object is return from an init method, we need to update self so that any subsequent instance variable references affect the right palces in memory." (实例变量位于距离self变量一个固定距离的内存区域 ，如果一个新的对象是由init方法返回的，我们需要更新self，使得每个子实例变量指向内存中正确的位置。)
 如果初始化错误，init方法会返回nil。``self = [super init]`` 保证了在 ``[super init]`` 返回nil时，不执行{}里的代码。
###init中要做的事
1. 立即加载。可以在init中完成对变量的内存分配和初始化；
2. 懒加载。可以在init中不构造变量。将变量的accessor方法写为懒加载方式，待需要时再调用。