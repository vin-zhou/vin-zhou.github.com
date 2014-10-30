title: Objective-C 之内存管理
date: 2014-10-30 15:34:17
tags:
---
## 引用计数 (Reference Counting)
Cocoa使用了引用计数（reference counting,也称为 retain counting)的技术来进行内存管理。每个对象都有一个integer成员变量，
即引用数(reference count 或 retain count)。

* reference count 加一的情况：
 * alloc/new
 * copy
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
当你创建一个自动释放池时，它将自动变为活跃的池。在Mac OSX 10.4之后，Objective-C提供了一个更好的-drain方法，其只清空池中的对象，但
并不会销毁池子。

**推荐使用第一种方法，因为其更快！**

### Cocoa的内存管理规则
* 如果你使用new, alloc, copy 使得 retain count值为1，那么你需负责
给其发送 release 或 autorelease 消息。
* 如果你使用了其他object的引用，假设其retain count 为1 且 将会被
autorelease即可。
* 如果你retain了一个object,你需要负责release或autorelease它。
* 
<table>
    <tr>
        <th>获得方式...</th>
        <th> 短期使用 </th>
        <th> 长期使用 </th>
    </tr>
    <tr>
        <td>由new, alloc, copy生成 </td>
        <td> 在使用完立即release   </td>
        <td> 在dealloc中release   </td>
    </tr>
     <tr>
        <td>其他方式</td>
        <td> 啥都不用做 </td>
        <td> 使用Retain获取，并在dealloc中release </td>
    </tr>
</table>

* 其他规则
