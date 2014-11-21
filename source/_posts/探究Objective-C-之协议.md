title: 探究Objective-C 之协议
tags: iOS
categories: 技术
date: 2014-11-20 17:01:42
---
Objective-C提供的协议（Protocol）类似于Java中的接口（interface）,提供了一种扩展方法和属性的方式。在采用了（adopt）协议后，需要实现协议中所有的方法，否则编译器会报warning。因此Protocol通常要根据功能，其声明的方法要少且有内聚性。
## 采用协议的语法

```objc
@interface Car : NSObject<NSCopying, NSCoding>
{}

// methods
@end // Car
```
**<>**内的即为2个协议, 其放置顺序无意义。