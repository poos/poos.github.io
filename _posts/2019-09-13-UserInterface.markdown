---
layout:     post                       # 使用的布局（不需要改）
title:      重回 Layout                  # 标题
subtitle:   Frame, UIViewAutoresizing, NSLayoutAnchor, Flexbox, SwiftUI, FlutterUI, VFL            #副标题
date:       2019-09-13                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 总结
---

### 相关链接


##### [Apple/Design/HumanInterfaceGuidelines](https://developer.apple.com/design/human-interface-guidelines/ios/overview/themes/)

##### [Texture/LayoutExample](https://texturegroup.org/docs/automatic-layout-examples-2.html)

##### [Xcode/InterfaceBuilder](https://developer.apple.com/xcode/interface-builder/)

##### [SwiftUI](https://developer.apple.com/xcode/swiftui/)

##### [Flutter/UserInterface](https://flutter.dev/docs/development/ui)

##### [AutoLayoutVisualFormat](https://www.raywenderlich.com/277-auto-layout-visual-format-language-tutorial#:~:text=The%20Auto%20Layout%20Visual%20Format,constraints%20one%20at%20a%20time)



在iOS上可用的布局方式大体分为 InterfaceBuilder 和 Code，而在 Code 方面因为 Frame 的不方便，衍生了很多 Layout 方式。那么就一起比对一下吧～

### Frame

Frame 是最简单粗暴的布局方式：```CGRect(x: 0, y: 0, width: 36, height: 36)```，那同时也是效率最高的布局方式。

使用不同的 Layout 方式，在复杂 TableView 的快速滚动上，使用 Model 提前计算 Frame，重用时候直接调整 Frame 且显示，往往能够获得更高的帧数。（注意是通常情况下，一些深度优化的多线程技术例外：Texture等）

上句提到的性能最佳，也是因为其他布局往往是在主线程占有资源，计算为 Frame 并最终使用的。

### UIViewAutoresizing (InterfaceBuilder / Code)

![img](https://poos.github.io/img/layout_1.png)

```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

如图，最简单的布局方式2，哈哈哈～

然鹅...其实并没有，这种布局往往只是相当于给 Frame 加了个 Extension，之后不用苦逼的计算每个屏幕下应该怎么扩展显示了。也就是说，如果你之前是有一套自己的 Frame 计算逻辑的话，这个好像很鸡肋，功能又不强大，迁移又太麻烦。

~~这么一说好像为它开一个栏的必要都木有了，~~继续默默的等比例，或者等边距，哈哈哈～


### NSLayoutAnchor (InterfaceBuilder / Code)

啊，早在 iOS9 的时候美丽的 NSLayoutAnchor 已经横空粗现了，你是否已经爱上了它...

![img](https://poos.github.io/img/layout_2.png)
![img](https://poos.github.io/img/layout_3.png)

讲实话这个确实是 Layout 的神器，搭配 Xib/Storyboard，使用起来不要太爽了～

- 边界/宽高约束
- Priority
- Text Size
- Scroll Content
- StackView

而且搭配 Code，能够实现所有你想要的布局，就是说纯 Code 能达到的，配合起来能更方便的达到。

![img](https://poos.github.io/img/layout_4.gif)


再来一个然鹅...因为这种布局书写起来真的比较麻烦，所以少见有全使用 Code 来添加约束的~~，也是因为大家的选择太多了，不必盯着Code这一棵树~~。

如果要使用这个方案，这一趴还是使用 Xib 或者 Storyboard 来用吧，更有利于项目的整体风格和代码观点～

下边的 Code 就简单看一眼吧：

```swift
extension UIView {

    /* UILayoutGuide objects owned by the receiver.
     */
    @available(iOS 9.0, *)
    open var layoutGuides: [UILayoutGuide] { get }
    ...
}

extension UIView {

    /* Constraint creation conveniences. See NSLayoutAnchor.h for details.
     */
    @available(iOS 9.0, *)
    open var leadingAnchor: NSLayoutXAxisAnchor { get }
    ...
}

open class NSLayoutAnchor<AnchorType> : NSObject, NSCopying, NSCoding where AnchorType : AnyObject {

    // NSLayoutAnchor conforms to <NSCopying> and <NSCoding> on macOS 10.12, iOS 10, and tvOS 10

    // These methods return an inactive constraint of the form thisAnchor = otherAnchor.
    open func constraint(equalTo anchor: NSLayoutAnchor<AnchorType>) -> NSLayoutConstraint
    ...
}

// Axis-specific subclasses for location anchors: top/bottom, leading/trailing, baseline, etc.

@available(iOS 9.0, *)
open class NSLayoutXAxisAnchor : NSLayoutAnchor<NSLayoutXAxisAnchor> {
    // A composite anchor for creating constraints relating horizontal distances between locations.
    @available(iOS 10.0, *)
    open func anchorWithOffset(to otherAnchor: NSLayoutXAxisAnchor) -> NSLayoutDimension
}

@available(iOS 9.0, *)
open class NSLayoutYAxisAnchor : NSLayoutAnchor<NSLayoutYAxisAnchor> {
    // A composite anchor for creating constraints relating vertical distances between locations.
    @available(iOS 10.0, *)
    open func anchorWithOffset(to otherAnchor: NSLayoutYAxisAnchor) -> NSLayoutDimension
}

// This layout anchor subclass is used for sizes (width & height).

@available(iOS 9.0, *)
open class NSLayoutDimension : NSLayoutAnchor<NSLayoutDimension> {
    // These methods return an inactive constraint of the form thisVariable = constant.
    open func constraint(equalToConstant c: CGFloat) -> NSLayoutConstraint
    ...
}
```

#### iOS11 以下拖入 WKWebview

当你在 Storyboard 使用 WKWebview 的时候，你会发现如果你的项目支持 iOS11 以下，那么拖入 WKWebview Xcode 会报错？纳尼？？

**尽管 WKWebview 从 iOS8 开始提供，但是在 –[WKWebView initWithCoder:] 有个bug，一直到 iOS11 才被修复！！**
[Apple document/WKWebview](https://developer.apple.com/documentation/webkit/wkwebview)
[WKWebView before iOS 11.0](https://bmnotes.com/2019/01/25/error-wkwebview-before-ios-11-0-nscoding-support-was-broken-in-the-previous-version/)

所以在使用的时候需要通过代码初始化创建，那么就需要用代码添加约束了～

```
    _webView = [WKWebView new];
    _webView.translatesAutoresizingMaskIntoConstraints = false;

    [_contentView addSubview: _webView];

    [[_webView.leadingAnchor constraintEqualToAnchor:_contentView.leadingAnchor] setActive:true];
    [[_webView.trailingAnchor constraintEqualToAnchor:_contentView.trailingAnchor] setActive:true];
    [[_webView.topAnchor constraintEqualToAnchor:_contentView.topAnchor] setActive:true];
    [[_webView.bottomAnchor constraintEqualToAnchor:_contentView.bottomAnchor] setActive:true];
```

### Flexbox

[Google/FlexboxLayout](https://github.com/google/flexbox-layout)

好的框架往往是可以移植的，而且大家是乐于去移植和使用的。Texture 就是借鉴了 Flexbox 的布局方式。

![img](https://poos.github.io/img/layout_5.png)

盒式布局使用的是盒子堆盒子、者盒子套盒子，所以有一些方面使用是非常舒服的～

- 布局共用，因为是以盒子为单位，所以更能促进大家公用Node，毕竟如果你不想公用就只能自己实现...
- Content 的适应，盒子有很多填充方式，用于适配不同屏幕甚至横屏
- StackView 布局等


> [Texture(ASDK)的理解和使用](https://poos.github.io/2019/02/26/Swifter/)，Swift 语言的新特性。那么也科普和安利一下，Texture 也就是 ASDK，自己实现了一套异步渲染框架，来给手机显示提供更高的性能。简言之就是使用一套与 View 匹配的 Node 来在背景线程计算布局，最终在主线程直接使用计算完成的值，这样来提供强大的性能。当然除了这个基本目标，它也使用了当时最先进的布局方式，FlexBox，写起来真的不要太爽。**在一些复杂table上性能提升最为明显**，在做内容类，商品类产品的时候，使用这套框架确实提升了相当的性能。

### SwiftUI / FlutterUI

如果做 Android 或者 RN 开发，常见的就是 xml 布局和逻辑代码的分离化。相反的如果你习惯了使用 Code 去码界面，那么这一扒一定不要错过！

那其实 FlutterUI 的布局方式就是借鉴的 FlexBox，这块可以和 FlexBox 一起看。紧随其后出现的 SwiftUI 可以 Code 方式跟 Flutter 几乎一样，等到大家的项目都是 iOS 13 起始，并且新建项目的时候就又多了个选择😂 。

```swift
ZStack {
    EmptyView().frame(width: 100, alignment: .bottom);
    Image("img").scaledToFill();
    Text("hello").font(.headline).foregroundColor(.red)
}
```

~~还是挺好玩的，等 iOS 13 全民时代的时候 Swift 也能 build 出 Android 就舒服了～~~ 所以个人认为，现在 Fultter 还是非常占优的！
> 2020/06/22 iOS 13的百分比已经占到了 87%

### VFL

全称是 Auto Layout Visual Format Language Tutorial, 应该是比较小众的布局方式了，毕竟需要手写 Format 字符串，小手一抖那可就不好了...

不过还是可以欣赏一下这种 Format 的思想，另外，现在 Swift可以支持自定义符号了，完全可以定义符号避免手写format。不过个人感觉这种满满的 Code 味道，不具备现代编程语言简单快速清晰等优点，这也是小众的原因吧。
```
// 1
let views: [String: Any] = [
  "iconImageView": iconImageView,
  "appNameLabel": appNameLabel,
  "skipButton": skipButton]

// 2
var allConstraints: [NSLayoutConstraint] = []

// 3
let iconVerticalConstraints = NSLayoutConstraint.constraints(
  withVisualFormat: "V:|-20-[iconImageView(30)]",
  metrics: nil,
  views: views)
allConstraints += iconVerticalConstraints

// 4
let nameLabelVerticalConstraints = NSLayoutConstraint.constraints(
  withVisualFormat: "V:|-23-[appNameLabel]",
  metrics: nil,
  views: views)
allConstraints += nameLabelVerticalConstraints

// 5
let skipButtonVerticalConstraints = NSLayoutConstraint.constraints(
  withVisualFormat: "V:|-20-[skipButton]",
  metrics: nil,
  views: views)
allConstraints += skipButtonVerticalConstraints

// 6
let topRowHorizontalConstraints = NSLayoutConstraint.constraints(
  withVisualFormat: "H:|-15-[iconImageView(30)]-[appNameLabel]-[skipButton]-15-|",
  metrics: nil,
  views: views)
allConstraints += topRowHorizontalConstraints

// 7
NSLayoutConstraint.activate(allConstraints)
```

### 总结

多种布局方式各有优劣，应当最大限度的遵循当前项目的风格，但是如果项目风格确实太老了，不妨逐步迁移使用新的。重要的是确定规则并严格的实施。
