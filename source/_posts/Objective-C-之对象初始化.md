title: Objective-C-之对象初始化
date: 2014-11-06 21:54:36
tags: iOS
categories: 技术
---
## 分配对象空间
Allocation: 给一个类发送alloc消息将会导致该类分配一块足够大的内存空间来存放其所有实例；同时内存会被初始化设为0.即：BOOL被设为NO； int 设为0；float 设为 0.0；指针设为 nil。
一个被alloc的对象并不能马上使用，还需要调用init方法。在C++、Java等语言中，对象的内存分配和初始化都被在一个构造函数中实现；Objective-C将其分为了2步。如果只使用alloc，没有init，则很可能出现奇怪的错误。
你应该在alloc后，链式使用init函数如：
```objc
  Car* car = [[Car alloc] init];
```
而不是
```objc
  Car* car = [car alloc]; [car init];
```
这是因为 init得到的对象可能和alloc得到的并不是同一个。在有些情况， 如 NSString 的init方法会根据不同的参数（非常长的字符串亦或是阿拉伯字母）来选择要构造的对象，并替代原来的对象。
