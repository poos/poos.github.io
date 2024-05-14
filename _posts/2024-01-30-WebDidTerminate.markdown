---
layout:     post
title:      WebView 调试
subtitle:   页面重载机制
date:       2024-01-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- Code
---


### 简介

web 调试可能碰到种种问题。那么如何处理调试呢？


#### 问题一、webViewWebContentProcessDidTerminate

选择了一个图片后，webview 重新加载了？

##### 背景
在这篇文章有详细的系统函数调用上下文、流程等。 [深入理解WKWebView白屏](https://blog.csdn.net/u012413955/article/details/89671301)

简单来说，系统会因为如下的不同的原因，进入不同的流程。具体是在内存超出、CPU超出、以及发生了Crash的场景下需要重新刷新（1次，内部是有一个计数）。其他情况下保持原状。

```
ProcessTerminationReason {
    ExceededMemoryLimit,//超出内存限制
    ExceededCPULimit,//超出CPU限制
    RequestedByClient,//主动触发的terminate
    Crash,//web进程自己发生了crash
    NavigationSwap,//m页的加载环境出现了变化
}
```

而开发者可以介入，就是在 webViewWebContentProcessDidTerminate 方法。但是遗憾的是这个方法没有传入 reason。

```
protocal WKNavigationDelegate {
    func webViewWebContentProcessDidTerminate(_ webView: WKWebView);
}
```
##### 简单深入
其实苹果是有一个私有方法，可以拿到reason。使用如下的示例代码。

```
class CWebview: WKWebView {
}

extension CWebview: WKNavigationDelegate {
    func webViewWebContentProcessDidTerminate(_ webView: WKWebView) {
        NSLog("-----17web ter: \(webView)")
        webView.reload()
    }
    @objc
    func _webView(_ webView: WKWebView, webContentProcessDidTerminateWithReason reason: NSInteger) {
        NSLog("-----22web ter \(reason): \(webView)")
        webView.reload()
    }
}

```
代码运行后发现：
- 优先 webContentProcessDidTerminateWithReason，这两个方法只会调用一个（也是符合一贯的风格）。
- 要声明 @objc 才能被调用到。


**如何调试**

有个文档可以非常有效的在模拟器进行测试：

使用 `kill $(pgrep -P $(pgrep launchd_sim) 'com.apple.WebKit.WebContent')` 即可触发移除web crash类型的异常。

详细的查看如下文档。

```
# In the command line, find the PID of your simulator process:
ps -p `pgrep launchd_sim`

# or if you have many simulators running:
ps -A | grep launchd_sim

# Find the PID of the WebContent process:
pgrep -P <simulator-pid> 'com.apple.WebKit.WebContent'

# kill it
kill <webcontent-pid>

# or if you want a one-lner:
kill $(pgrep -P $(pgrep launchd_sim) 'com.apple.WebKit.WebContent')
```


[模拟器，从命令行终止 Web 内容进程以触发调用webViewWebContentProcessDidTerminate](https://qa.1r1g.com/sf/ask/4994502901/)


真实设备上的话，暂没找到可以触发方式。可预见的是OOM类型的可以通过其他app配合调试。


**一点思考，为啥 WKNavigationDelegate 不用显式声明 @objc 呢？**

尝试一下几个解法：
1、如上 @objc ，可以
2、如下 @objc ，可以

```
@objc
extension CWebview: WKNavigationDelegate, CWKNavigationDelegate {
    func _webView(_ webView: WKWebView, webContentProcessDidTerminateWithReason reason: NSInteger) {
        NSLog("-----22web ter \(reason): \(webView)")
        webView.reload()
    }
}

@objc
protocol CWKNavigationDelegate {
    func _webView(_ webView: WKWebView, webContentProcessDidTerminateWithReason reason: NSInteger);
}
```
3、如下 @objc ，不可以

```
@objc
extension CWebview: WKNavigationDelegate, C2WKNavigationDelegate {
    func _webView(_ webView: WKWebView, webContentProcessDidTerminateWithReason reason: NSInteger) {
        NSLog("-----22web ter \(reason): \(webView)")
        webView.reload()
    }
}

@objc
protocol CWKNavigationDelegate {
}

protocol C2WKNavigationDelegate {
    func _webView(_ webView: WKWebView, webContentProcessDidTerminateWithReason reason: NSInteger);
}
```

4、参考 WKNavigationDelegate，继承自 NSObjectProtocol，不可以

看来纯纯就是要声明一下 @objc，其他花里胡哨的都没有用。@objc也不支持传递。 protocal 也不能用 final， 所以，这个场景咱还是直接加 @objc 在函数上保险。
而系统的可以不用声明，大概率是编译器做了适配。
