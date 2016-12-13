title: 探究Objective-C-之runtime
tags: iOS
categories: 技术
date: 2015-08-26 10:18:16
---
主要参考自：[南峰子](http://southpeak.github.io/blog/2014/10/25/objective-c-runtime-yun-xing-shi-zhi-lei-yu-dui-xiang/) 和 [王中周](http://blog.csdn.net/wzzvictory/article/details/8615569)的个人博客, 提取了其中感兴趣的点。

##前言
作为一门动态编程语言，Objective-C 会尽可能的将编译和链接时要做的事情推迟到运行时。只要有可能,Objective-C 总是使用动态 的方式来解决问题。这意味着 Objective-C 语言不仅需要一个编译环境,同时也需要一个运行时系统来执行编译好的代码。**运行时系统（runtime）扮演的角色类似于 Objective-C 语言的操作系统,Objective-C 基于该系统来工作。因此，runtime好比Objective-C的灵魂,很多东西都是在这个基础上出现的。Objc Runtime其实是一个Runtime库，它基本上是用C和汇编写的，这个库使得C语言有了面向对象的能力。**
##要点

* Class
* objc_object 与id
* objc_cache
* Meta Class（元类）
  当我们向一个对象发送消息时，runtime会在这个对象所属的类的方法列表中查找方法；而当我们向一个类发送消息时，会在这个类的meta-class的方法列表中查找。meta-class是一个类对象所属的类，每个类都会有一个单独的meta-class，因为每个类的类方法基本不可能完全相同。
  
##类与对象操作函数
runtime提供了大量的函数来操作类和对象。类的操作方法大部分以class为前缀，而对象的操作方法大部分是以objc或object_为前缀。

Parameter |  说明
------ |-------
const char* class_getName(Class cls); | 获取类的类名
Class class_getSuperclass(Class cls); | 获取父类
BOOL class_isMetaClass(Class cls);    | 是否是元类
size_t class_getInstanceSize(Class cls);|获取实例大小
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char* types); | 添加方法
Method class_getClassMethod(Class cls, SEL name); | 获取类方法
Method* class_getInstanceMethod(Class cls, SEL name); | 获取实例方法
IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char* types); | 替换方法的实现
....|....
objc_alllocateClassPair(Class superclass, const char*name, size_t textraBytes);               | 创建一个新类和元类
objc_registerClassPair(Class cls); | 注册创建的类
Ivar object_getInstanceVariable(id obj, const char* name, void** outvalue);|获取对象实例变量的值
Ivar object_setInstanceVariable(id obj, const char* name, void* value); | 修改对象实例变量的值
id object_getIvar(id obj, Ivar ivar); | 返回对象中实例变量的值
const char* object_getClassName(id obj) | 获取对象的类名
Class object_getClass(id obj); | 获取对象的类
Class object_setClass(id obj, Class cls); | 设置对象的类
....|....
## 方法与消息
###SEL
Objective-C在编译的时候，会根据方法的名字，生成一个用来区分这个方法的唯一的一个ID，这个ID就是SEL类型的,本质上就是一个char型指针。如下所示：
```objc
typedef struct objc_selector *SEL;  //定义
------
SEL sel1 = @selector（method1）;
NSLog(@"sel : %p", sel1);
NSLog(@"%s", (char*)selector); 
上面的输出为:
2015-8-23 18:40:07.518 RuntimeTest[52734:466626] sel : 0x100002d72
015-8-23 18:40:07.518 RuntimeTest[52734:466626] method1
```
我们需要注意的是，只要方法的名字相同，那么它们的ID都是相同的。就是说，不管是超类还是子类，不管是有没有超类和子类的关系，只要名字相同那么ID就是一样的。当然，不同的类可以拥有相同的selector，这个没有问题。不同类的实例对象执行相同的selector时，会在各自的方法列表中去根据selector去寻找自己对应的IMP。
工程中的所有的SEL组成一个Set集合，Set的特点就是唯一，因此SEL是唯一的。因此，如果我们想到这个方法集合中查找某个方法时，只需要去找到这个方法对应的SEL就行了，SEL实际上就是根据方法名hash化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度上无语伦比！！但是，有一个问题，就是数量增多会增大hash冲突而导致的性能下降（或是没有冲突，因为也可能用的是perfect hash）。但是不管使用什么样的方法加速，如果能够将总量减少（多个方法可能对应同一个SEL），那将是最犀利的方法。那么，我们就不难理解，为什么SEL仅仅是函数名了。
本质上，SEL只是一个指向方法的指针（准确的说，只是一个根据方法名hash化了的KEY值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。
我们可以在运行时添加新的selector，也可以在运行时获取已存在的selector，我们可以通过下面三种方法来获取SEL:

1. sel_registerName函数
2. Objective-C编译器提供的@selector()
3. NSSelectorFromString()方法

###IMP
IMP实际上是一个函数指针，指向方法实现的首地址,其定义如下：
```objc
id (*IMP)(id, SEL, ...)
```
第一个参数是只想self的指针（如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针），第二个参数是方法选择器(selector)，之后是方法的实际参数列表。根据SEL取得IMP后，我们就获得了执行这个方法代码的入口点，此时，我们就可以像调用普通的c语言函数一样来使用这个函数指针了。

通过取得IMP，我们可以跳过Runtime的消息传递机制，直接执行IMP只想的函数实现，这样省去了Runtime消息传递过程中所做的一系列查找操作，会比直接向对象发送消息高效一些。

###Method
```objc
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name                 OBJC2_UNAVAILABLE;  // 方法名
    char *method_types                  OBJC2_UNAVAILABLE;
    IMP method_imp                      OBJC2_UNAVAILABLE;  // 方法实现
}  
```
Method用于表示类定义中的方法，可以看上其包含了一个SEL和IMP，相当于在SEL和IMP之间做了一个映射。
###方法相关操作函数
包括以下：
```objc
// 调用指定方法的实现
id method_invoke ( id receiver, Method m, ... );

// 调用返回一个数据结构的方法的实现
void method_invoke_stret ( id receiver, Method m, ... );

// 获取方法名
SEL method_getName ( Method m );

// 返回方法的实现
IMP method_getImplementation ( Method m );

// 获取描述方法参数和返回值类型的字符串
const char * method_getTypeEncoding ( Method m );

// 获取方法的返回值类型的字符串
char * method_copyReturnType ( Method m );

// 获取方法的指定位置参数的类型字符串
char * method_copyArgumentType ( Method m, unsigned int index );

// 通过引用返回方法的返回值类型字符串
void method_getReturnType ( Method m, char *dst, size_t dst_len );

// 返回方法的参数的个数
unsigned int method_getNumberOfArguments ( Method m );

// 通过引用返回方法指定位置参数的类型字符串
void method_getArgumentType ( Method m, unsigned int index, char *dst, size_t dst_len );

// 返回指定方法的方法描述结构体
struct objc_method_description * method_getDescription ( Method m );

// 设置方法的实现
IMP method_setImplementation ( Method m, IMP imp );

// 交换两个方法的实现
void method_exchangeImplementations ( Method m1, Method m2 );
```
### 消息调用流程
一切还是从消息表达式[receiver message]开始，在被转换成objc_msgSend(receiver, SEL)后，在运行时，runtime system会做以下事情：

1、检查忽略的Selector，比如当我们运行在有垃圾回收机制的环境中，将会忽略retain和release消息。

2、检查receiver是否为nil。不像其他语言，nil在objective-C中是完全合法的，并且这里有很多原因你也愿意这样，比如，至少我们省去了给一个对象发送消息前检查对象是否为空的操作。如果receiver为空，则会将 selector也设置为空，并且直接返回到消息调用的地方。如果对象非空，就继续下一步。

3、接下来会根据SEL到当前类中查找对应的IMP，首先会在cache中检索它，如果找到了就根据函数指针跳转到这个函数执行，否则进行下一步。

4、检索当前类对象中的方法表（method list），如果找到了，加入cache中，并且就跳转到这个函数之行，否则进行下一步。

5、从父类中寻找,直到根类：NSObject类。找到了就将方法加入对应类的cache表中，如果仍未找到，则要进入：[动态方法决议](http://blog.csdn.net/wzzvictory/article/details/8629036)。

6、如果动态方法决议仍不能解决问题，只能进行最后一次尝试，进入消息转发流程。

7、如果还不行，去死吧。

####动态方法解析
由NSObject根类提供的类方法，调用时机为 当被调用的方法实现部分没有找到，而消息转发机制启动之前的这个中间时刻：
```objc
+(BOOL) resolveInstanceMethod:(SEL) sel
+(BOOL) resolveClasMethod:(SEL) sel
---------------------

// 举例：
void dynamicMethodIMP(id self, SEL _cmd)
{
    // implementation ....
}

+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    NSLog(@"sel is %@", NSStringFromSelector(sel));
    if(sel == @selector(setName:)){
        class_addMethod([self class],sel,(IMP)dynamicMethodIMP,"v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```
####消息转发
对象是谦恭的，它会接收所有发送过来的消息，哪怕这些消息自己无法响应。问题来了：当对象无法响应这些消息时怎么办？runtime提供了消息转发机制来处理该问题。

当外部调用的某个方法对象没有实现，而且resolveInstanceMethod方法中也没有做重定向处理时，就会触发- (void)forwardInvocation:(NSInvocation *)anInvocation方法。在该方法中，可以实现对不能处理的消息做的一些默认处理,也可以以其它的某种方式来避免错误被抛出。像forwardInvocation:的名字一样,这个方法通常用来将不能处理的消息转发给其它的对象。通常我们重写该方法的方式如下所示
```objc

-(void)forwardInvocation:(NSInvocation *)invocation
{
	SEL invSEL = invocation.selector;
	if ([someOtherObject respondsToSelector:invSEL])
		[anInvocation invokeWithTarget:someOtherObject];
	} else {
		[self doesNotRecognizeSelector:invSEL]; 
	}                                                                          
}
```

##参考
* [Objective-C中的@dynamic](http://www.csdn123.com/html/blogs/20131019/85539.htm)
