title: Objective C 拾遗
date: 2015-03-18 10:15:16
tags: iOS
categories: 技术

---
本文补充了一些在使用Object C时见到的狗皮膏药，主要是OOAP方面。

## public private protected 方法的实现

由于Object C不支持对方法使用访问修饰符：

You can neither declare a method protected or private. Objective-C's dynamic nature makes it impossible to implement access controls for methods. (You could do it by heavily modifying the compiler or runtime, at a severe speed penalty, but for obvious reasons this is not done.)

因此，需要使用一种work around方法：

###public 方法
在.h文件中声明的方法为public方法
###private方法
在.m文件的上部，用unnamed category声明的方法为private方法
###protected
如同 UIGestureRecognizerSubclass, 使用Category将需要protected的方法和property封装起来：

```objc
//  UIGestureRecognizerSubclass.h
@interface UIGestureRecognizer(UIGestureRecognizerProtected)

@property(nonatomic, readwrite) UIGestureRecognizerState state;

-(void)reset;
.....
```
这样只有添加``UIGestureRecognizerSubclass.h``头文件的类，才能通过继承访问其protected方法。