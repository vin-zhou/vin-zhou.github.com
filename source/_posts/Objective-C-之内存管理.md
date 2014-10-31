title: Objective-C 之内存管理
date: 2014-10-30 15:34:17
tags:
---
## 引用计数 (Reference Counting)
Cocoa使用了引用计数（reference counting,也称为 retain counting)的技术来进行内存管理。每个对象都有一个integer成员变量，
即引用数(reference count 或 retain count)。

* reference count 加一的情况：
 * alloc/new/copy
 * 发送 retain 消息给对象, 即调用 -（id) retain函数， 如[car retain] ;

* 减一的情况：
 * 发送 release 消息给对象, 与retain类似，调用 -（oneway void) release 函数, 如[car release]；
 

* 当count值=0时，将会自动给该对象发送一个dealloc消息，调用其dealloc函数，将其占用的内存返还给系统。千万不要直接调用dealloc，
而应该由Objective-C来调。
 
## 对象所有权（Object Ownership) 
一个对象包含指向其他对象的成员变量，则该对象对所指对象拥有所有权。    
例如，对于Car对象，包含一个Engine* engine;成员变量

```C
    @interface Car
    {
        Engine* engine;
    }
    @end
```
对于engine的accessor函数，则应该如此设置：

```c
   - (void) setEngine: (Engine *) newEngine
   {
        [newEngine retain];
        [engine release];
        engine = newEngine;
   }
```
 即在accessor函数中，需要先retian新对象，再release旧对象，防止当newEngine与engine为同一对象时出现错误。
 
## Autorelease
NSObject 提供了一个方法：

```c
 - (id) autorelease;
```
这个方法预定了在将来的某个时刻会调用release方法。当你发送一个autorelease消息给一个object时，实际上会将该object添加到 autorelease 池中，当这个池销毁时，池中的所有对象才会
收到release消息。即相当于在 NSAutorleasePool 类的dealloc函数中，将会给池中的每个对象都发送一个release消息。  
例如：

```c
- (NSString *) description
{
    NSString *description = [[NSString alloc] initWithFormat:@"I am %d years old", 4];
    return ([dscription autorelease]); // 而不是 return (description).
}
```
### 那么何时创建一个autorelease池呢？     
有两种方法：

1. 使用 @autoreleasepool   
  这样，所有在
```c
 @autoreleasepool
{     
}
```
  中的代码，都会被该池处理。对于内存非常敏感的计算，还可以嵌套。**需要注意的是，所有在{}内的变量，不能在之外再使用。**
* 使用 NSAutoreleasePool 对象    
例如：   
```c
NSAutoreleasePool *pool = [NSAutoreleasePool new];
...
[pool release];
```
**当你创建一个自动释放池(autorelease pool)时，它将自动变为活跃的池。autorelease 采用类似栈的形式进行存储，即新建的pool将
置于栈顶，为活跃的栈，之后的autorelease消息会将接收对象放置于栈顶的池中。**
在Mac OSX 10.4之后，Objective-C提供了一个更好的-drain方法，其只清空池中的对象，但并不会销毁池子。

**推荐使用第一种方法，因为其更快！**

### Cocoa的内存管理规则
* 如果你使用new, alloc, copy 使得 retain count值为1，那么你需负责
给其发送 release 或 autorelease 消息。
* 如果你使用了其他object的引用，假设其retain count 为1 且 将会被
autorelease即可。
* 如果你retain了一个object,你需要负责release或autorelease它。



获得方式 | 短期使用 | 长期使用
------------ | ------------- | ------------
由new, alloc, copy生成 | 在使用完立即release  | 在dealloc中release
其他方式 | 啥都不用做  | 使用Retain获取，并在dealloc中release


* 举例：

```c
    NSMutableArray * array = [[NSMutableArray alloc] init]; // alloc, 需要手动release或autorelease
    [array release];
```
```c
    // 非alloc/new/copy，因此无需考虑release
    NSMutableArray * array = [NSMutableArray arrayWithCapcity:17]; 
    NSColor *color = [NSColor blueColor];
    id object = [someArray objectAtIndex: i]; 
```

### 保持pool干净
一个例子：

```c
int i;
for (i = 0; i< 10000000; i++)
{
    id object = [someArray objectAtIndex: i];
    NSString *desc = [object descrption];
    // and do something with the description
}
```
这种写法方式只会在整个循环后，autorelease pool再释放各desc字符串; 下面是改进的写法，手动每1000子释放一次pool:
```c
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
int i;
for (i = 0; i< 10000000; i++)
{
    id object = [someArray objectAtIndex: i];
    NSString *desc = [object descrption];
    // and do something with the description
    
    if(i % 1000 == 0)
    {
        [pool release]; // 主动释放
        pool = [[NSAutoreleasePool alloc] init]; // 新建
    }
}
[pool release];
```

## Automatic Reference Counting, ARC
* 与桌面系统不同，移动端需要更确切的垃圾回收时间，因此苹果对iOS提出了**自动引用计数（ARC)**来处理资源回收问题；当使用ARC后，可以正常的创建和使用object，由编译器在适当的地方补上retains和releases。
* ARC不同于GC，GC是在运行时执行的，依据代码的执行，定期/不定期的检查；而ARC在编译时处理，补上retains和releases，系统并不
关心这些代码是由手写的还是由ARC补全的。

### ARC只对3类 retainable object pointers(ROPs)起作用：
  1. Block 指针；
  * Objective-C 对象指针；
  * \__attribtue__((NSObject))标记的typedef  .
  
 所有其他的指针如 char*, CF对象如 CFStringRef，malloc c array等，都不是ARC支持的，需要自己手动处理。

### 使用ARC 需满足的3个条件
1. 首先需要判断对象是否属于之前3类的ROPs;
* 需指出如何retain、release对象，即大部分需继承自NSObject；
* 提供传递对象ownership的方式。

### 有时使用weak reference更合适
* **强引用**：当你的对象参与了所指对象的内存管理，称之为 strong reference
* **弱引用**：如果不参与，称之为 weak reference, 弱引用可防止 retain cycles。例如在A中使用 @property(assgine) B* b; 
 B的引用计数加1， A的引用计数不会加1（强引用则都要加1），此时release A，由A release B就可以了。

* 对于弱引用，如果所指的对象已经被释放了，则系统会自动清理这个引用。
例如：

```c
   __weak NSString* myString;
   @property(weak) NSString* myString;
```

使用ARC时要注意：

* property 不能一new开头，如 @property NSString *newString 是非法的( **会报：`Property follows Cocoa naming convention for returning 'owned' objects 的错误**)； 
### Ownership 拥有优先权
