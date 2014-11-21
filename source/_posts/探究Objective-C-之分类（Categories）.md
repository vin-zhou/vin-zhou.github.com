title: 探究Objective-C 之分类（Categories）
tags: iOS
categories: 技术
date: 2014-11-14 16:32:14
---
Objective-C提供了category机制来支持给已有类添加新方法。
其作用主要有3点：
1. 将类的实现分散到多个不同文件或多个不同框架中
2. 创建私有方法的前向引用
3. 向对象添加非正式协议

## 将类的实现分散到多个不同文件或多个不同框架中
category的基本语法：
```objc
@interface ClassName(CategoryName)
// method declarations
- (XX *) methodName;
@end // ClassName+CategoryName.h

-------------------------

@implementation ClassName(CategoryName)
- (XX *) methodName
{
  .......
} // ClassName+CategoryName.m
```
表示给类ClassName添加一个类别CategoryName。 通常将category放到独立的文件中，命名为 *ClassName+CategoryName*。
可以给category添加@prorperty,但不能添加实例变量，且property必须是@dynamic标注的（在.m中）。使用@property可以简化setter/getter的编写，并支持用.访问。

当然，也可以将类别的声明放到已有的其他.h文件中，将其实现放到其他.m文件中。
### 特殊的类别： class extension（类扩展）
其特点在于：
1. CategoryName为空；
2. 可以添加实例变量；
3. 可以将变量由readonly改为readwrite;
4. 可以有任意多个。（但容易出bug，慎用）
例如：

```objc
@interface Things: NSObject
@property (assign)NSInteger thing1;
@property (readonly, assign) NSInteger things2;
- (void)resetAllValues;
@end // Things.h
```
可以使用类扩展，定义如下：
```objc
@interface Things()
{
 NSInteger thing4; // extension
}
@property(readwrite, assign)NSInteger thing2; // overwrite
@property(assign)NSinteger thing3; //extension
@end // Things.m
```
可对信息隐藏的类进行扩展。**注意：类扩展是直接在原始的.m文件中添加的，并不会添加独立的.h和.m文件。**
### 分散.m文件内容
可以使用category对大的类实现文件.m进行分割，将其分散到多个.m文件中。（也利于团队分工合作开发）
例如：
  Appkit中的NSWindow包含了数百个方法，使用category，其按逻辑被分割到多个文件中：
  * @interface NSWindow(NSKeyboardUI)
  * @interface NSWindow(NSToolbarSupport)
  * @interface NSWindow(NSDrag)
  * @interface NSWindow(NSCarbonExtensions)
  
再如 NSString, 其在Foundation framework被声明，包含了文本内容；但其category NSStringDrawing 放在Appkit，用于实现在屏幕上的绘制方法， 绘制string的文本。

## 对私有方法前向引用
在Cocoa中，没有真正的私有方法，如果你知道了一个类的私有方法名称（即使没有在@interface中的声明）， 你都可以直接调用该方法，不过会报 "warning instance method not found"。为了合法使用，可以使用category进行前向引用。

例如，类TestObj有私有函数calcNum：方法：
```objc
@interface TestObj : NSObject
@end // TestObj.h
----------------------------
@implementation TestObj
-(int) calcTireNum    // 私有方法
{
    return 4;
}
@end // TestObj.m
```
可以在category的TestObj+Num.m文件中声明后，文件内直接访问:
```objc
@interface TestObj(Num)
- (void) printTireNum; // 建立新方法，不暴露旧方法
@end // TestObj+Num.h
-------------------------
@interface TestObj()  // 使用类扩展
- (int) calcTireNum; // 声明旧的私有方法，获得其访问权限
@end
@implementation TestObj(Num)
- (void) printTireNum
{
    NSLog(@"%d", [self calcTireNum]); // 可以自由地访问私有方法了
}
@end  // TestObj+Num.m
```

## 充当非正式协议（Informal protocol）

Cocoa中大量使用了代理（delegate），比如NSTableView负责处理了所有的滑动列表（scrolling lists）， 当tableView在处理时（比如选择用户点击的行），tableView将询问其代理来查看该行是否可以选择。tableView给其代理发送一个信息：
```objc
- (BOOL) tableView:(NSTableView *)tableView shouldSelectRow:(NSInteger)rowIndex;
```
由该代理方法根据tableView和rowIndex来查看该行是否应该被选中。

* NSNetServiceBrowser
 NSNetServiceBrowser提供了网络服务，可以给服务设置代理，在发现需要的服务时browser对象会回调代理对象。
 
 
* NSRunLoop
一个run loop是一个Cocoa结构，当关注的事情（如net service browser发现新的服务时）发生时，才开始执行。
run loop不仅能监听网络流通情况，还可以等待用户事件，如键盘按键、鼠标点击等。run方法将不会返回，一直执行，因此其之后的代码将永远不会执行。

###非正式协议
非正式协议是指给NSObject添加类别，声明一系列方法。这样任意的对象都可以通过只实现类别中感兴趣的方法来充当代理对象，而不必实现所有。

### selector
@selector只是一个编译标记，表示对方法名字进行特殊编码，以供Objective-C运行时快速查找。
如：
```objc
@selector(setEngine:)
@selector(setTire:atIndex:) 
```
NSObject提供了一个方法：``respondsToSelector:``, 可以检查出对象是否对给定的消息进行响应(即是否包含了该方法）。
例如：
```objc
Car *car = [[Car alloc] init];
if ([car respondsToSelector: @selector(setEngine:)])
{
	NSLog(@"can response !");
}
```

selector 还可以作为参数传递到方法中，或是保存到变量中。 selector相关详细知识点可以参看[探究Objective-C之消息机制](/2014/11/09/探究Objective-C-之属性)


## category的缺点

* 不能添加变量实例；
* 命名冲突：category里的方法会覆盖原类中的同名方法；（可以通过加特定前缀来避免）

## 参考
* 《Learn Objective C on the Mac 2nd Edition》