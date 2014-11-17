title: 探究Objective-C 之Categories
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
可以给category添加@prorperty,但不能添加实例变量，且property必须是@dynamic标注的。使用@property可以简化setter/getter的编写，并支持用.访问。

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
@end
```
可以使用类扩展，定义如下：
```objc
@interface Things()
{
 NSInteger thing4; // extension
}
@property(readwrite, assign)NSInteger thing2; // overwrite
@property(assign)NSinteger thing3; //extension
@end
```
可对信息隐藏的类进行扩展。
### 分散.m文件内容
可以使用category对大的类实现文件.m进行分割，将其分散到多个.m文件中。（也利于团队分工合作开发）
例如：
  Appkit中的NSWindow包含了数百个方法，使用category，其按逻辑被分割到多个文件中：
  * @interface NSWindow(NSKeyboardUI)
  * @interface NSWindow(NSToolbarSupport)
  * @interface NSWindow(NSDrag)
  * @interface NSWindow(NSCarbonExtensions)
  
再如 NSString, 其在Foundation framework被声明，包含了文本内容；但其category NSStringDrawing 放在Appkit，用于实现在屏幕上的绘制方法， 绘制string的文本。

## 使用category进行前向引用
在Cocoa中，没有真正的私有方法，如果你知道了一个类的私有方法名称（甚至没有在@interface中的声明）， 你就可以通过category调用该方法。


## category的缺点

* 不能添加变量实例；
* 命名冲突：category里的方法会覆盖原类中的同名方法；（可以通过加特定前缀来避免）