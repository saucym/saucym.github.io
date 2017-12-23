---
title: iOS实现横竖屏遇到的一些坑和一些使用技巧
categories: iOS
tages: [orientation,转屏,pop动画异常]
---
项目中频繁的使用一些转屏，开发和维护过程中遇到很多坑，这里把转屏的基本流程和使用技巧分享一下。

一、首先介绍一下转屏的基本流程。
    1.设备硬件加速计检测到方向变化的时候会发出UIDeviceOrientationDidChangeNotification通知。
    2.app收到通知会询问AppDelegate的window支持的方向情况与window的rootViewController或rootViewController.presentedViewController(如果存在)所支持的方向做交集，然后完成旋转。
二、旋转控制函数有下面几个
     - (BOOL)shouldAutorotate;// 将要自动旋转 返回 yes 为支持旋转，no 为不支持旋转  (这里有个坑，后面会讲)
     - (UIInterfaceOrientationMask)supportedInterfaceOrientations;// 支持的界面方向
     - (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation;// 首选方向，刚present进去时的方向

三、下面是两个实践例子
例子(1).先来一个简单的，一个只支持Portrait的界面A present 一个支持三个方向(UIInterfaceOrientationMaskAllButUpsideDown)的界面B，并且首选方向是左
这样很简单，在A中写两个函数

```Objective-C
- (BOOL)shouldAutorotate {
    return NO;
}
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskPortrait;
}
```

    在B中写三个函数就搞定了
```Objective-C
- (BOOL)shouldAutorotate {
    return YES;
}
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskAllButUpsideDown;
}
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return UIInterfaceOrientationLandscapeLeft;
}
```
    然后就可以直接看效果了
[图片](presentB)
    这时候你会发现在界面B旋转屏幕到非Portrait后dismiss会crash，因为dissmiss的时候window会询问A的方向，和当前window方向做交集，交集为空就crash了，解决办法很简单，在dismiss之前转回A页面支持的方向就可以了，代码如下
```Objective-C
- (instancetype)init {
    self = [super init];
    if (self) {
        self.originDeviceOrientation = [UIDevice currentDevice].orientation;
    }
    return self;
}
- (void)dismissViewControllerAnimated:(BOOL)flag completion:(void (^)(void))completion {
    [[UIDevice currentDevice] setValue:@(self.originDeviceOrientation) forKey:@"orientation"];
    [super dismissViewControllerAnimated:flag completion:completion];
}
```

例子(2).再来一个常用的，一个只支持Portrait的界面A push 一个支持三个方向(UIInterfaceOrientationMaskAllButUpsideDown)的界面B
    要实现这个能力除了在A、B控制器里面分别实现上面写的函数外还需要转发转屏控制，因为window只询问到UINavigationController或UITabBarController，我们这里需要对UINavigationController的消息做转发，代码如下

```Objective-C
@interface UINavigationController(Autorotate)
@end

@implementation UINavigationController(Autorotate)

- (BOOL)shouldAutorotate {
    return self.viewControllers.lastObject.shouldAutorotate;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return self.viewControllers.lastObject.supportedInterfaceOrientations;
}

- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return self.viewControllers.lastObject.preferredInterfaceOrientationForPresentation;
}

@end
```
    但是这样会出现一个bug，如果B处于横屏状态pop，然后再次push到B在pop，动画就会异常。类似下面这样的异常动画
    查了半天没找到原因，然后发现手Q没这个问题，于是查看手Q代码发现手Q里面没有实现shouldAutorotate这个函数，其实supportedInterfaceOrientations这一个函数就完全可以搞定屏幕方向了，不知道Apple的工程师为什么设计两个接口来控制屏幕方向。这里把shouldAutorotate函数干掉就没问题了。

我们项目里面就有一个(2)这样的使用场景，前前后后遇踩了好多坑，总感觉这种使用方式有问题。最近思考了下，并把Apple本家的app都下载下来使用了下，没有看到(2)这种使用方式，而且系统API询问转屏只询问到UINavigationController说明Apple的API设计里面可能都没有考虑(2)这种使用场景。最后总结了下转屏的几个注意事项，欢迎大家批评指正。
    1.全局都不用写shouldAutorotate，只写supportedInterfaceOrientations就够了。
    2.不要像例子(2)里面一样去分发UINavigationController或UITabBarController的转屏控制函数，如果做了就要做好踩坑的准备，实在产品需求没办法可以考虑用present并定制一个跟push一样的transition动画。
    3.支持不同方向的界面要像例子(1)这样用present，因为官方文档里面都是说的这样使用。
