---
layout:     post                       # 使用的布局（不需要改）
title:      "记录一次 Webview 迁移的过程" # 重回 Layout                  # 标题
subtitle:   协议制定，Storybaord 修改，类修改         #副标题
date:       2020-10-21                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
- Code
---

### 背景

在5个月之前就有收到相关的消息，要更新 UIWebview 到 WKWebview。虽然苹果的最后期限一直往后推，但是不可避免的项目修改终于来了。

整个修改要说多也不多，但是还是单独开了一篇文章，因为一些细节还是值得一提的。

背景的的噱头比较大：

- 古老的 Main.storyboard 有 1M +
- 古老的 Main.storyboard 有 ScrollView 使用了 contentLayoutGuide 和 frameLayoutGuide，但是这两个特性在 Xcode 的版本适应问题
- 古老的 Main.storyboard 打开即有超百项修改。
- 使用了 UIWebview 有 5 个 以上地方，且有可能有使用多种方式。


问题有一个结：在 Storyboard 的 Webview怎么处理，解决好这个结问题就简单多了。

基本上整个流程是：

- 修复 contentLayoutGuide 和 frameLayoutGuide 的问题
- 创建通用的协议
- 使用协议修改项目工程
- 善后




#### 关于源码xml
为了解决这个结，必须要看看 storyboard 里面的代码是什么样的。


可以看到，是一份 UTF-8 的 xml 文件。


而我们最关心的 webView 就是以标签的样式存在着，这就给了我们直接去源码修改的可能。

> 还能看到其他的参数，delegate 之类...

```xml
<?xml version="1.0" encoding="UTF-8"?>
...
<webView translatesAutoresizingMaskIntoConstraints="NO" contentMode="scaleToFill" id="4sF-t2-MpG">
    <rect key="frame" x="5" y="0.0" width="310" height="500"/>
    <color key="backgroundColor" white="0.0" alpha="0.0" colorSpace="calibratedWhite"/>
    <connections>
        <outlet property="delegate" destination="T1J-I6-9wj" id="60D-a7-u0y"/>
    </connections>
</webView>
...
```

#### contentLayoutGuide 和 frameLayoutGuide

`error: Content and frame layout guides before iOS 11.0`

因为 xx 某个提交引入了这个错：在使用 Scrollview 的时候，使用了 contentLayout。

当项目在 Xcode 11.3 的时候运行没有问题，但是当项目在 Xcode 11.4+ 运行的时候即发生了错误。

> 就是 Xcode 11.3 也应该报这个错的，但是没有报错，可能属性被忽略了，但是没有给出错误提醒。


这个问题究其根本还是 Code 的写法有问题，所以直接进入 1M + 的 storyboard修改了即可。


其实最终就是在 storyboard 去掉 contentLayoutGuide 和 frameLayoutGuide 的相关属性描述。

```xml
<viewLayoutGuide key="contentLayoutGuide" id="39H-sL-yCR"/>
<viewLayoutGuide key="frameLayoutGuide" id="1kj-R0-kSP"/>
```


#### 创建通用的协议 / View

这段属于精髓之一了。为了能让代码有最小的修改，创建了一个继承 WKWebView 的类，用于桥接旧的 Webview delegate。


很简单， 仿照 UIWebView 的代理方法进行一层封装，然后提供一个替换 UIWebview 的 UI 类。
```swift
#import <UIKit/UIKit.h>
#import <WebKit/WebKit.h>
NS_ASSUME_NONNULL_BEGIN
@protocol SXWebViewDelegate <NSObject>
@optional
- (BOOL)webView:(WKWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(WKNavigationType)navigationType;
- (void)webViewDidStartLoad:(WKWebView *)webView;
- (void)webViewDidFinishLoad:(WKWebView *)webView;
- (void)webView:(WKWebView *)webView didFailLoadWithError:(NSError *)error;
@end
@interface SXWKWebView : WKWebView
@property (weak, nonatomic) id<SXWebViewDelegate> delegate;
@end
NS_ASSUME_NONNULL_END
```

```swift
#import "SXWKWebView.h"
#import <WebKit/WebKit.h>
@interface SXWKWebView () <WKNavigationDelegate>
@end
@implementation SXWKWebView
- (instancetype)initWithCoder:(NSCoder *)coder
{
    self = [super initWithFrame:CGRectZero configuration:[WKWebViewConfiguration new]];
    if (self) {
        self.translatesAutoresizingMaskIntoConstraints = NO;
    }
    return self;
}
#pragma mark - WKNavigationDelegate
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    if (_delegate == nil) {
        decisionHandler(true);
    }

    BOOL result = [_delegate webView:webView shouldStartLoadWithRequest:navigationAction.request navigationType: navigationAction.navigationType];
    decisionHandler(result);
}
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {
    [_delegate webViewDidStartLoad: webView];
}
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    [_delegate webViewDidFinishLoad: webView];
}
- (void)webView:(WKWebView *)webView didFailNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    [_delegate webView:webView didFailLoadWithError: error];
}
@end
```


#### 修改 SXWKWebView 以支持 Storyboard 拖入 delegate


1. 属性添加 IBOutlet 标志

`@property (weak, nonatomic) id<SXWebViewDelegate> delegate;`

->

`@property (weak, nonatomic) IBOutlet id<SXWebViewDelegate> delegate;`

2. 实现 Set 方法

```
- (void)setDelegate:(id<RSHWebViewDelegate>)delegate {
    _delegate = delegate;
    self.navigationDelegate = self;
}
```

#### 修改 Storyboard 和 Code

1. Storyboard 只需要修改标签到`<webView ...> </webView>` 到 `<view ...> </view>` 即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
...
<view translatesAutoresizingMaskIntoConstraints="NO" contentMode="scaleToFill" id="4sF-t2-MpG">
    <rect key="frame" x="5" y="0.0" width="310" height="500"/>
    <color key="backgroundColor" white="0.0" alpha="0.0" colorSpace="calibratedWhite"/>
    <connections>
        <outlet property="delegate" destination="T1J-I6-9wj" id="60D-a7-u0y"/>
    </connections>
</view>
...
```

2. 代码只需要修改`UIWebView` 到 `WKWebView` 即可。

```swift
- (void)webViewDidStartLoad:(WKWebView *)webView{
    [_progressService presentBusyIndicator];
}
- (void)webViewDidFinishLoad:(WKWebView *)webView
{
    [_progressService dismissBusyIndicator];
}
```

至此，项目就可以愉快的跑起来了。给 QA 出一个测试包吧～

#### 善后


**1. 如果使用了 Pod 或者其他包管理工具的问题。**

**2. 网页实际效果问题**

有些网页在 UIWebView 上显示很好，但是在 WKWebView 显示被缩放很小。评论说，WKWebView 更接近手机 Safari 的显示效果。
// https://stackoverflow.com/questions/26102908/suppress-wkwebview-from-scaling-content-to-render-at-same-magnification-as-uiweb

不过，为了缩放正常，我们可以添加 `initial-scale=1.0` 来进行控制。

```swift
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation {
    NSString *javascript = @"var meta = document.createElement('meta');meta.setAttribute('name', 'viewport');meta.setAttribute('content', 'width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no');document.getElementsByTagName('head')[0].appendChild(meta);";
    [webView evaluateJavaScript:javascript completionHandler:nil];
}
```


### 总结

这些是使用过程中，一些方便的点，希望能够帮助到看到这篇文章的人。
