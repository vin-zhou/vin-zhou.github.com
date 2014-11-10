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
两种方式：
1. 立即加载。可以在init中完成对变量的内存分配和初始化；
2. 懒加载。可以在init中不构造变量。将变量的accessor方法写为懒加载方式，待需要时再调用。

###提供便捷的init方法
许多对象提供了多种以init开头的方法，这类方法约定为初始化方法（initializer），且返回类型只能是``(id)``类型的。如NSString 提供了：
1. -(id) init;
2. -(id) initWithFormat:(NSString*) format,...;
3. -(id) initWithContentOfFile:(NSString*)path encoding:(NSStringEncoding)enc error:(NSError**)error;
例如：
```objc
NSError* error = nil;
NSStringEncoding encoding = NSUTF8StringEncoding;
NSString* string = [[NSString alloc] initWithContentsOfFile:@"/tmp/words.txt"
usedEncoding:&encoding
error:&error];
if(nil != error)
{
  NSLog(@"Unable to read data from file, %@", [error localizedDescription]);
}
```
### designated initializer (特定的初始化方法)
类中有一个init方法被设为 designated initializer。所有这个类的initalizer方法都使用了这个方法来完成初始化工作。子类也使用这个designated initializer方法来为父类进行初始化。拥有参数最多的init方法通常被认为是 designated initializer。在使用其他人的代码时，需要确定哪个init方法是designated initialzier。
例如, 对于Tire的各init方法：

```objc
 - (id）init;
 - (id) initWithPressure: (float)pressure;
 - (id) initWithTreadDepth: (float) treadDepth;
 - (id) initWithPressure: (float)pressure treadDepth:(float)tp;
```
由于initWithPressure:treadDepth:参数最多, 因此可以被用做为指定的初始化方法：

```objc
- (id) init
{
	if(self = [self initWithPressure:34 treadDepth:20])
    {}
    return (self);
} // init
- (id) initWithPressure: (float) p
{
  if(self = [self initWithPressure:p treadDepth:20])
    {}
    return (self);
} // initWithPressure
- (id) initWithThreadDepth: (float) td
{
  if(self = [self initWithPressure:34 treadDepth:td])
    {}
   return (self);
} // initWithThreadDepth

// 子类 AllWeatherRadial的初始化方法
- (id) initWithPressure:(float) p treadDepth(float) td
{
   if(self = [super initWithPressure:p treadDepth: td])
   {
	rainHanding = 23;
    snowHanding = 42;
   }
   return (self);
} // initWithPressure:treadDepth
```
## 初始化方法的准则
* 不必一定有自定义的initializer方法。当alloc默认的清零结果满足需要时，不必再写init方法；
* 如果自定义了初始化方法，一定先调用父类的designated init方法。
* 如果自定义的init方法有多个，则需要选一个作为designated initializer。该方法需调用父类的designated initializer方法，且该类其他的init方法也要调用这个designated initializer方法。

## dealloc方法
在dealloc方法中，需要先释放当前类的成员变量，最后调用父类的dealloc方法，如：

```objc
- (void) dealloc
{
[tires release];
[engine release];
[super dealloc];
} // dealloc
```
需要保证最后一句是 ``[super dealloc]``。
在garbage-colleced 中， 则没有dealloc方法；如果你需要在对象释放的时候做一些工作，则可以重写 ``finalize``。 ARC与之相似，使用了@autoreleasepool来处理retain和release周期。
