---
layout: post
title: "适配iOS7上的状态栏和导航栏"
date: 2014-10-14 10:03:19 +0800
comments: true
categories: iOS
---

[原文链接](http://blog.jaredsinclair.com/post/61507315630/wrestling-with-status-bars-and-navigation-bars-on-ios-7)

在为 *Riposte* 和 *Whisper* 这两个应用适配iOS7系统时，我反复遇到的一个问题是如何让应用的视图层级能够正确布局。iOS7中最麻烦的API就是那些为系统状态栏和 `UINavigationController` 提供的。在撰写本文时，这方面的API文档是严重不足。下文罗列了这方面的一些必要知识，给那些还在和iOS7的状态栏和视图控制器苦苦纠缠的程序员。
  
1. 没有什么方法可以保留iOS6风格的状态栏布局，也就是说，系统状态栏始终会覆盖在你的iOS7应用上。 

2. 不要把状态栏的外观(appearance)和状态栏的布局(layout)混淆起来。外观（light或者default）并不会影响到状态栏如何布局（frame/height/overlap）。同样需要注意的一点是系统的状态栏不再有任何的背景色。当API使用 `UIStatusBarStyleLightContontent` 时, 它指的是白色文字显示在一个透明的背景上。`UIStatusBarStyleDefault` 则指的是黑色文字显示在透明的背景上。  

3. 状态栏的外观由两个互斥的方式控制：你可以通过代码的方式设置，或者由 `UIKit` 根据 `UIViewController` 中一些新的属性更新，其中默认的后面这种。检查你应用中 ***ViewController Based status Bar Appearance*** 对应的plist值来确认你当前使用的是哪一种方式。如果这个值是 `YES` ，那么你应用中每一个顶层视图控制器（不是那些容器视图控制器）需要重写 `preferredStatusBarStyle()` ，返回default或者是light样式。如果你设置了这个值为NO，那么你就可以使用你熟悉的 `UIApplication` 方法去控制状态栏的外观。
4. `UINavigationController` 会根据一个很诡异的约束来判断 `UINavigationBar` 的高度应该是44点还是64点。如果 `UINavigationController` 检测到其视图的顶部和 `UIWindow` 的顶部是相连的，那么它将导航栏绘制为64点的高度。如果其视图的顶部和UIWindow的顶部并不相连（哪怕只是一个点的间隔），那么它将导航栏绘制为“传统”的44点的高度。`UINavigationController执` 行这个逻辑即使这个视图控制器有多个子控制器。没有什么办法可以去避免这种行为。
    
5. 如果你给一个自定义的导航栏设置了一个只有44点高度的背景图，并且 `UINavigationController` 中视图的 `bounds` 和 `UIWindow` 的 `bounds` 一样（正如第4点中讨论的），那么 `UINavigationController` 将会在（0, 20, 320, 40）的位置绘制你的导航栏背景图，并在这个背景图的上方留下20点的黑色不透明空间。这个结果也许会让你觉得自己是一个聪明的程序员，成功绕过了第1点中所说的规则。但其实你错了，导航栏的高度仍旧是64点。  

6. 请小心使用 `UIViewController` 的 `edgesForExtentLayout` 属性。调整 `edgesForExtendedLayout` 在大多数情况下不会有任何效果。`UIKit` 使用这个属性的唯一方式是如果你添加一个 `ViewController` 到 `UINavigationController`，这个 `UINavigationController` 会使用该属性判断其子视图控制器是否显示在导航栏或者状态栏下面。当给 `UINavigationController` 设置 `edgesForExtendedLayout` 并不会改变 `UINavigationController` 是有44还是64高度的导航栏高度。参考第4点中的处理逻辑。类似的布局逻辑同样适用于使用在工具栏或者 `UITabBarController` 的视图底部。  

7. 如果你要实现的仅仅是防止你的子视图控制器显示在导航栏的底部。那么将 `edgesForExtendedLayout` 设置为 `UIRectEdgeNone`(或者至少是一个不包含`UIRectEdgeTop`的Mask)，请在这个 `UIViewController` 的生命周期内尽早设置这个值。  

8. `UINavigaionController` 以及 `UITabBarController` 会尝试给Table View或者Collection Views类型的子视图添加contentInsets属性值。 这个和第4点中所提到的状态栏处理逻辑类似。有一个代码处理方式可以去避免这个情况，设置 Table View和Collection Views的 `automaticallyAdjustsScrollViewInsets` 为 `NO` （缺省为 `YES` ）。这个属性之前给 **Riposte** 和 **Whisper** 带来了一些严重问题，因为我之前是通过`contentInset`属性去控制因键盘以及工具栏移动而导致的Table View的布局改变。

9. 重申一点：并没有回归到iOS6样式的状态栏布局逻辑的方法。为了实现这个相似的效果，你可以将应用中所有的视图控制器(译者注: 应该是视图控制器中的视图)移至一个和屏幕顶部有20像素偏移的容器视图内，并在状态栏的后面留一个黑色的视图。这就是最后我在 **Riposte** 和 **Whisper** 中的使用的方法。  

10. 苹果一直设法确保你不会去尝试第9点中的方法，他们希望我们能够重新设计APP使其能正确显示在状态栏下。有充足的论据，无论是用户体验还是技术原因，都说明了为什么这终究不是一个好方法。 你需要做那些对用户最好的，而不是简单的遵循平台的诡异特性。
