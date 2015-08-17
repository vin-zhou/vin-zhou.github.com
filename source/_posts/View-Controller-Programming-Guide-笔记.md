title: View Controller Programming Guide 笔记
tags: iOS
categories: 技术
date: 2015-01-12 09:35:20
---

1. 不要给Window添加直接添加View， 而是添加 view controller，controller关联的View将会自动被添加上。这是因为controller会自动管理view在需要时被加载，在特定条件下被释放，即controller起到了重要的资源管理的作用。
2. 每个View都只被一个view controller 管理，每个view controller处理部分数据，因此，controller间需要communicate with other以显示整个功能。
3. View controller 通常可被划分为两类：content view contrllers 和 container view controllers。
  3.1  content view controller
  在同一个view 层次结构中， content view controller和其管理的view要是一对一的。
   （1） table view controller
  
  3.2 container view conroller
  container view controller包含了由其他view controllers所拥有的content。这些view controllers 显示地成为了container的孩子。contianer间也可构成父子关系。由此也构成了一个view controller的层次结构。
  container view contrller同时也可以像content view controller一样， 添加view（由其子controller所拥有）到自己的view层次结构上container来决定何时添加以及size如何。
   （1）Navigation conroller
   （2）Tab bar contoller
   （3）Split view controller
   （4）Page view controller

4. Communcation between source and destination view controllers（控制流模型）
   * 由目标视图控制器提供properites供源视图控制器设置传参；
   * 如果目标视图控制器想给源视图控制器传参，则通常使用代理。首先在目标视图控制器中定义一个delegate成员，由源视图控制器实现该代理的协议，然后将自己赋值给目标视图控制器的变量，再由目标视图控制器通过该变量传值给源视图控制器。
   * 这样做的好处是， 目标视图控制器由源视图控制器来驱动，并尽可能地较少获知源视图控制器的内容（只知道代理方法）。因此提高了可复用性。
 
5. 使用storyboard
  segue的类型：
  * push segue: 将目标视图控制器push到navigation控制器栈中；
  * modal segue: present目标视图控制器；
  * popover segue: 在一个popover中显示目标视图控制器；
  * custom segue: 自定义

6. Storyboard就像在讲一个story，其上的controller多是通过segue触发或child-container关系存在的，构成了一幅连通图，因此在segue被触发或parent container被实例化之后才实例化；也有些controller没有连接在图上，其通过代码被调用和实例化。可以为controller和segue添加identifier string来唯一标识它。标识符对于通过代码被调用的segue和controller是必须的, 如下方法：
  ``performSegueWithIdentifier:sender:``。

7. 可以创建一个storyboard来维护所有的controller，也可以创建多个storyboard，其中一个通常被一直定义为主storyboard，iOS来自动加载（可以在配置中设置），其他的storyboard必须在程序中被显示地用代码来加载, 如``[UIStoryboard storyboardWithName:bundle:]``。

8. 当主storyboard被设置后，程序加载时，iOS执行以下操作：
   1. 初始化一个window;
   2. 加载主storyboard并初始化其initial view controller;
   3. 将此controller设置伟window的rootViewContoller，使window visible.

9. 手动实例化一个storyboard中的view controller:
   1. 获得storyboard。如果该view controller与其他已加载的controller在同一个storyboard上，则可通过其.storyboard属性来获得storyboard;否则需使用UIStoryboard的类方法``storyboardWithName:bundle:``，传递文件名和可选的bundle参数来创建；
   2. 调用storyboard的``instantiateViewControllerWithIdentifer:``方法来加载在Interface Builder设定indentifier的该controller;或是``instantiateInitialViewController``来直接加载initial view controller；
   3. 设置这个新的view controller的属性
   4. 显示这个view controller, 有以下几种方式可供选择：
     * 设置其为一个window的root view controller
     * 设置其为一个visible contrainer view controller的child
     * 用另一个view contrller的present 方法将其显示出来
     * 使用popover将其显示出来（仅适用于iPad）
     
```objc 
// 加载一个新的storyboard
- (UIWindow*) windowFromStoryboard:(NSString*)storyboardName onScreen:(UIScreen*) screen
{
	UIWindow *window = [[UIWindow alloc] initWithFrame:[screen bounds]];
    window.screen = screen;
    
    UIStoryboard* storyboard = [UIStoryboard storyboardWithName:storyboardName bundle:nil];
    MainViewController* mainViewController = [storyboard instantiateInitialViewController];
    window.rootViewController = mainViewController;
    
    // configure the new view controller here.
    
    
    return window;
}
```

10. 管理ViewControllers的资源
   1. 区分所使用的各对象的生命周期，尽量使用懒加载；
   2. 当系统内存不足时，所有的 view controllers 将会自动手动收到一个 low-memory 警告, 因此需要在 view controllers 做好在 ``didReceiveMemoryWarning`` 方法中释放资源的准备.
   
   * Initializing a view controller
   在初始化时，view controller 需只创建或加载需要贯穿整个生命周期的资源和对象，而不能创建view hierarchy或任何显示内容相关的对象。需在初始化方法中只关注用来实现关键行为的对象和数据。如果使用Interface Builder，archive 将会在 ``initWithCoder:`` 方法中加载，然后任何实现了 ``awakeFromNib`` 方法的对象，该方法将会被调用；如果通过program创建view controller，init方法将会被调用。Init 方法应该尽量简单，尽量在 ``loadView`` 方法中再对其 properties 进行配置。
   * 在loadView方法中，view controller负责实例化 self.view 及 view hierarchy的 创建；然后在viewDidLoad方法中，可以得到已经创建好的view。如果使用auto layout来进行布局，应该实现 ``viewWillLayoutSubviews``和``viewDidLayoutSubviews``方法来控制subviews在view hierarchy中的position 和 frames.

11. ViewController调整subviews的操做
When the size of a view controller’s view changes, its subviews are repositioned to fit the new space available to them. The views in the controller’s view hierarchy perform most of this work themselves through the use of layout constraints and autoresizing masks. However, the view controller is also called at various points so that it can participate in the process. Here’s what happens:
	1. The view controller’s view is resized to the new size.
	2. If autolayout is not in use, the views are resized according to their autoresizing masks.
	3. The view controller’s viewWillLayoutSubviews method is called.
	4. The view’s layoutSubviews method is called. If autolayout is used to configure the view hierarchy, itupdates the layout constraints by executing the following steps:
		a. The view controller’s updateViewConstraints method is called.
		b. The UIViewController class’s implementation of the updateViewConstraints method calls the view’s updateConstraints method.
		c. After the layout constraints are updated, a new layout is calculated and the views are repositioned.
	5. The view controller’s viewDidLayoutSubviews method is called.
12. 
