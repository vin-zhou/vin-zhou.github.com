title: Swift编程札记
tags: Swift
categories: 技术
date: 2016-07-22 10:18:16
---

##要点


* 正则表达式

学习网站：http://deerchao.net/tutorials/regex/regex.htm
验证网站：https://www.debuggex.com/

* String 转 Double 在Objc 和 Swift中的不同

Objc
```objc
 NSString* lStr = @"34,467";
 double lDouble = lStr.doubleValue; // 值为 34
 lStr = @"34fd467";
 lDouble = lStr.doubleValue; // 值为 34
```
Swift
```objc
 var lStr = "34,467";
 if let lDouble = Double(lStr) {} // 值为 nil
 lStr = "34fd467";
 if let lDouble = Double(lStr) {} // 值为 nil
```
结论： Objc中NSString转double并不安全，Swift则安全。

##参考
* [Objective-C中的@dynamic](http://www.csdn123.com/html/blogs/20131019/85539.htm)
