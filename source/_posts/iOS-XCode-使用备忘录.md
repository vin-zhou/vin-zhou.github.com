title: iOS/Xcode 使用备忘录
date: 2014-11-03 17:26:47
tags: [iOS, Xcode]
categories: 技术
---
1. 设置main函数输入参数：
  Product->EditSceme->Arguments
  ![图片](/img/launch.png)
2. 调试时，可以直接在console中输入：
* 查看变量： ``po 变量名 ``
* 查看属性： ``po (int)[item retainCount]``
* 查看数组： ``po [array objectAtIndex:index]``
