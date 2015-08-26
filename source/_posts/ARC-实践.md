title: ARC 实践
tags: iOS
categories: 技术
date: 2015-02-02 08:53:55
---
## 总原则
  ARC只适用于Objective-C对象，不适用于C, Core Foundation， malloc(), free().因此仍需手动管理这些内容。

## 4个ARC必须遵守的规则：
1. No Access to Memery Methods
 * 不要使用 retain/release/autorelease..
 * 不要自定义以上方法
 * 解决方法：交给编译器来做这些
2. No Object Pointers in C Structs
 * 编译器必须知道references来自哪里去向何处
  	* 指针必须被初始化为0（0初始化）
    
    * 指针必须在reference goes away时被Release
  
   * 解决方法： 使用Objective-C中的对象
3. No Casual Castring id <-> void*
 * 编译器必须知道void*是否被retained
 * 提供了如下所示的Objective-C和Core Foundation-style 对象转换的新API：
 * Core Foundation 对象简称CF对象, 如 Core Graphics、Core Text， ARC环境下编译器不会自动管理CF对象的内存，所以当我们创建了一个CF对象后，需要CFRelease将其手动释放。因此CF和OC相互转化时，也需要使用特定转换符指定内存释放的责任。
 ```objc
 // old 
 CFStringRef w = (__bridge CFStringRef)A;
 NSString *x = (__bridge NSString*)B;
 CFStringRef Y = (CFStringRef)CFBridgingRetain(obj);
 NSString *z = (NSString*)CFBridgingRelease(ref);
 // new API
 CFStringRef w = (__bridge CFStringRef)A;
 NSString *x   = (__bridge   NSString*)B;
 CFStringRef Y = (_bridge_retain CFStringRef)obj;
 NSString *  z = (__bridge_transfer NSString*)ref;
 ```
 
 
  * `__bridge` 表示不改变ownership。如`id obj = (__bridge id)CFDictionaryGetValue(cfDict, key); `
  

  * `__bridge_retained` 作用同CFBridgingRetain, 表示将Ob-C对象的ownership 转交给CF对象，由我们自己维护生命周期。如
  `CFStringRef value = (__bridge_retained CFStringRef)[self someStrhing]; UseCFStringValue(value)；CFRelease(value);`

  * `__bridge_transfer` 作用同CFBridgingRelease，与__bridge_retained, 用于将CF对象的ownership转交给Ob-C 对象，之后由系统自动负责管理生命周期。如
  `NSString *value = (__bridge_transfer NSString*)CFPreferencesCopyAppValue(CFSTR("SomeKey"), CFSTR("com.company.someapp"));`
  使用之后，不必再重复调用CFRelease()，因为已经release一次了。

4. No NSAutoreleasePool
 * 编译器必须知道autoreleaed所控制的指针
 * NSAutoreleasePool 不是一个真正的对象，因此无法被retianed
 * 解决方法： 使用@autoreleasepool{  }.

## How do I think about ARC
* 考虑ownership:
 * strong 引用keep objects alive
 * Objects deallocated when no more strong refernces remain
 
* 考虑object graph
* **不再使用 retain/release/autorelease**
 
### Weak References
* Safe, nonretaining refernce
	* Drop to nil automatically(when object starts deallocation)
* Now supporte in ARC:
```objc
	@property (weak) NSView *myView; // property attribute
    __weak id myObject;              // variable keyword
```

### Strong References
  NSString *name; // same as  __strong NSString *name;
  * Default for all variables
  * Like a retain property
  
### Unsafe References
 __unsafe_unretained NSString *unsafeName = name;
  * Like a traditional variable, or an assign property
  * not initialized
  * no extra logic
  * no restrictions
  * useful in global structs with constant @"..." strings

### Object Graph Cycles
通过
 * set property values to nil
 * use weak reference
 
### Method Families
 * Naming convention
 * First "wrod" in first part of selector
 * **alloc , copy, init, mutableCopy, new ** transfer ownership
 * Everything else does not
 
## Convert to Objective-C ARC
* Remove all calls or implement to **retain, release, autorelease, retainCount**
* Replaces NSAutoreleasePool with @autoreleasepool
* @property(assign) becomes @property(weak) for objects pointers, remove retain, use default strong
* 可以保留dealloc的实现，用来释放资源，但不能调用[super dealloc]; 因其已被自动调用。
* 可以仍使用CFRetain, CFRelease with Core Fundation Objects，因为编译器不会自动管理Core Fundation 对象的生命周期
* 不能使用 NSAllocateObject or NSDeallocateObject
* 不能在C 数据结构中使用object pointers， 使用Objective-C class 取代这样的数据结构
* 不能使用memory zone
* 属性的name不能以new开头
* 对文件采用ARC, 加编译指令  `-fobjc-arc`
* 若不采用ARC, 可加编译指令 `-fno-objc-arc`

