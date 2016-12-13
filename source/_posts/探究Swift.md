title: 探究Swift
tags: Swift
categories: 技术
date: 2016-01-26 10:18:16
---
主要参考自：The Swift Programming Language.

##前言

##要点

## 基本语法

1. println("hello, world") 每行语句末尾不需要加分号
2. 使用let 声明常量（只能赋值一次）， var声明变量，声明时类型是可选的，如： let explicitDouble:Double = 70
3. 值永远不会隐式转换，除非你显示进行：
```swift
let label = "a"
let width = 9
let labelWidth = label + String(width)
 or 
let labelWidth2 = label + "\(width)"
```
4. 有些变量的值是可选的，一个可选的值可能是一个具体的值或者是nil， 表示值缺失。在``类型``后面加一个问号来标记这个变量的值是可选的：
```swift
var optionalString:String? = "hello"
optionalString = nil
```
5. for, if, while 用法
```swift
let interestingNumbers =  [
	"Prime":[2, 3, 4, 7],
    "Fibo": [1,1, 2, 3],
    "Square": [1, 3, 4, 5]
]
var largest = 0
for (kind, numbers) in interestingNumbers
{
	for number in numbers   // 不用括号
    {
    	if number > largest // 不用括号
        {
        	largest = number
        }
    }
}
// while---------
var n = 2
while n < 100
{
	n = n*2
}
等价于
var n;
for i in 2..100  // 或者 2...99 ，..创建的范围不包含上届， ...包含
{
	n = n*2
}
```

6. switch里的 case值可以是任意类型的数据以及各种比较操作，不一定是整数以及测试相等。也不用加break。
```swift
let vegetable = "red pepper"
switch vegetable
{
case "celery":
	let vegetableComment = "Add some .."
case "cucumber", "watercress"
    let vegetableComment = "That ..."
case let x where x.hasSuffix("pepper"):
	let vegetableComment = "Is it ..."
default:
	let vegetableComment = "Everything ..."
}
```
7. 函数和闭包
```swift
funct greet(name:String, day:String) -> String
{
	return "Hello \(name), today is \(day)."
}
greet("Bob" "Tuesday")
//使用元组返回多个值
func getGasPrices() -> (Double, Double, Double)
{
	return (3,59,5.7)
}
getGasPrices()
// 参数数量可变
func sumOf(numbers: Int ...) -> Int
{
	var sum = 0
    for number in numbers
    {
    	sum += number
    }
    return sum
}
sumOf()
sumOf(42, 597, 12)
```
函数是一等公民，因此函数可以作为另一个函数的返回值
```swift
func makeIncrementer() -> (Int -> Int) // 函数类型作为返回值类型
{
	func addOne(number:Int) -> Int
    {
    	return 1+number
    }
    return addOne // 返回函数
}
var increment = makeIncrementer()
increment(7)
```
函数也可以作为参数传入另一个函数
```swift
func hasAnyMatches(list: Int[], condition: Int -> Bool) -> Bool
{
	for item in list
    {
    	if condition(item)
        {
        	return true
        }
    }
    return false
}
func lessThanTen(number: Int) -> Bool
{
     return number < 10
}
var numbers = [20,  19, 7, 12]
hasAnyMatches(numbers, lessThanTen)
```
函数实际上是一种特殊的闭包，可以使用{}来创建一个匿名闭包， 使用in 来分割函数并返回类型
```swift
numbers.map(
{
	(number: Int) -> Int in
    	let result = 3*number
        return result
}
)
```
有很多种创建闭包的方法。如可以通过参数位置而不是参数名字来引用参数，当一个闭包作为最后一个参数传给一个函数的时候，它可以直接跟在括号后面：
```swift
sort([1,5,3,12,2]){$0 > $1}
```
8. 对象和类



##参考
* [Objective-C中的@dynamic](http://www.csdn123.com/html/blogs/20131019/85539.htm)
