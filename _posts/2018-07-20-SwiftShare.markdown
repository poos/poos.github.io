---
layout:     post                       # 使用的布局（不需要改）
title:      Swift多处调用分享的设计方案                 # 标题 
subtitle:   使用协议清晰明了的设计分享              #副标题
date:       2018-09-09                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- Swift
- 设计
---

> 当一个页面包含了太多的太杂的内容页面就变得复杂了，把相似的内容抽出来单独管理，让项目更清晰。


### 本文讨论不同页面调用分享的设计方案

某种意义上，不同页面调用分享也可以理解为继承重载的关系--->而继承重载可以用协议完美解决

#### 协议 类、结构体和枚举
[参考资料](https://blog.csdn.net/Lirx_Tech/article/details/41724185)

- 其实就是Java中的接口，Swift的遵守协议就是Java中的实现接口，如果放在C++中就是纯虚类的概念，即协议就是一种高度抽象的抽象类，里面值规定了方法、属性等内容，但没有提供任何实现，所有遵守该协议的类、结构体或枚举都必须实现协议中规定的内容，只不过C++没有接口而只能通过继承虚基类来实现其中的内容而已；

- 协议是面向接口编程的必不可少的机制，其要求定义与实现分离，在Java和Swift中都可以将协议作为数据类型（就和其它普通的数据类型Int、Array等）暴露给使用者，而使用者不用关心具体的实现细节；

- 协议中可以定义的内容：计算属性（实例/静态）、方法（实例/静态）和下标

#### 实现

这里实例调用分享的几个地方

```
//可能是 VC
class A {
    let title = "A"
    
    func beginShare() {
        self.share()//调用协议方法
    }
}
//可能是 View
class B {
    let title = "B"
    
    func clickAction() {
        self.share()//调用协议方法
    }
}

//可能是 WebView
class C {
    
    func jsShare() {
        self.share()//调用协议方法
    }
    
    func jsShareMoment() {
        self.shareMoment()//调用协议方法
    }
}
//可以是任何类
```

这里是协议分发管理的 protocol类
```
// 定义统一的协议
protocol ShareProtocol {
    var shareTitle: String { get }
    var shareUrl: String { get }
    func share()
}
protocol ShareProtocol2 {
    func shareMoment()
}

extension A: ShareProtocol {
    var shareTitle: String {
        return title + " share"
    }
    var shareUrl: String {
        return "https://baseurl/" + title
    }
    func share() {
        Share(title: shareTitle, url: shareUrl)
    }
}

extension B: ShareProtocol {
    var shareTitle: String {
        return title + " share"
    }
    var shareUrl: String {
        return "https://baseurl/" + title
    }
    func share() {
        Share(title: shareTitle, url: shareUrl)
    }
}
extension C: ShareProtocol, ShareProtocol2 {
    var shareTitle: String {
        return title + " share"
    }
    var shareUrl: String {
        return "https://baseurl/" + title
    }
    func share() {
        Share(title: shareTitle, url: shareUrl)
    }
    func shareMoment() {
        Share(title: shareTitle, url: shareUrl)
    }
}

```

### 总结

通过这种设计，ShareManager更加彻底的负责所有的分享事宜；VC 只是简单调用，页面结构更简单了。

