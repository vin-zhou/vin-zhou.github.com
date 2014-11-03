title: Objective-C 之异常处理
date: 2014-11-03 14:31:00
tags: iOS
---
首先在Xcode的build选项中，需将 *Enable Objective-C Exceptions* 选项选为 *Yes*.

## Exception关键字

* @try, @catch, @finally, @throw

```objc
@try
{
 // 可能会抛出异常的代码
 // 使用 @throw或 raise 一个NSException对象
}
@catch(NSException* exception)
{
 // 异常的处理代码
}
@finally
{
 // 不论是否有异常，总是会被执行的代码，通常用于 cleanup, 如release.
}
```

##捕捉不同的异常

```objc
 @try
 {}
 @catch(MyCustomException* custom)
 {}
 @catch(NSException* exception)
 {}
 @catch（id value)
 {}
 @finally
 {}
```
## 抛出异常
抛出一个NSException实例的方式有2种：
1. 使用@throw
2. 给一个NSException对象发送raise消息
例如：
```objc
// 先创建一个异常对象
NSExcetpion * theException = [NSException exceptionWithName:...];
// 然后可以通过 
 @throw theException; // 将其抛出 
或
 [theException raise]; // 发送异常消息
```
  注意：@try抛出异常不耗资源，然而捕获异常需要耗费资源和速度，因此不能用异常机制作为流程控制和错误处理。
## 异常和自动释放池（Autorelease Pool）
例如：

```objc
- (void) myMethod
{
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSDictionary* myDictionary = [[NSDictionary alloc] initWithObjectsAndKeys:@"asdfasds", nil];
    @try
    {
        [self processDictionary:myDictionary];
    }
    @catch (NSException *exception)
    {
        @throw; // 可再次抛出该异常
    }
    @finally
    {
        [pool release]; // cleanup
    }
}
```
此时在@catch中抛出的异常，将会在@finally执行完之后再处理，因此由于资源池已被释放了，造成该异常 对象被释放，**再次抛出时出错**。
改进方法：

```objc
- (void) myMethod
{
    id savedException = nil;
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSDictionary* myDictionary = [[NSDictionary alloc] initWithObjectsAndKeys:@"asdfasds", nil];
    @try
    {
        [self processDictionary:myDictionary];
    }
    @catch (NSException *exception)
    {
        savedException = [e retain]; 
        @throw; // 再次抛出该异常
    }
    @finally
    {
        [pool release]; // cleanup
        [savedException autorelease];
    }
}
```
**这里通过先将异常retain，并且这个异常被设为autoreleasse，这样在执行@finally之后，该异常可以仍可被正确的rethrow，并在适当的时候自动释放。**