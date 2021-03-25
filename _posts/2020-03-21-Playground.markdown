---
layout:     post
title:      那些年写过的 Demo～
subtitle:   Animaiton, Storyboard, Framework, Quene, Leak...
date:       2020-03-21
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Code
- Repo
---

### 前

列一下写过的 Demo，总结和备查

纯干货篇～


Demo 网址：

[poos/SomeDemo](https://gitee.com/poos/SomeDemo)


#### 2分钟制作数字动画

**NumberAnimation**

使用 StackView 包装数字0-9；添加动画即可。


#### 测试 Target Framework

**TestAppForTargetFramework**

在 target 提供BaseUI相关信息。

```
public extension UIColor {
    static var primary: UIColor { UIColor.red }
}

extension String {
    static public var name: String { "Name" }
}
```


#### Strory Tips

**StoryboardTips**

Storyboard 引用其他view。

[Stroyboard Tips](https://poos.github.io/2020/11/21/StoryboardTips/)


#### Synchorinized 使用 with 封装

**Synchorinized**

DispatchQueue sync 简单封装

```
func with(quene: DispatchQueue, f: () -> Void) {
    quene.sync {
        f()
    }
}
```


#### Swift 扩展给 OC代码调用

**TestExtensonSwift**

开放代码给swift；然后用swift写扩展；然后OC调用即可。


- TestExtensonSwift-Bridging-Header.h
- TestViewController.swift `@objc`


#### 测试日期处理

**TestCurrentIsToday**

测试一些日期的函数。


#### 测试 Stack View 布局

**TestLayout**

使用 StackView 布局复杂的UI。

还有 hide 和 switch 动画的测试。


#### 测试 IBDesignable 在 Xcode 12.3+ 报错的问题。

**TestIBDesignable**

项目可以正常跑～

> 问题原因是相关 framework 中引用的库没有被正确的使用xcframework配置。

使用 StackView 布局复杂的UI。

BaseUI 库测试。

测试 IBDesignable 是否能正常更新。


#### 测试线程安全的数组和字典

**TestLockArrAndDic**

使用 DispatchQueue 进行线程约束。

[Swift 的 Array 和 Dictionary 源码欣赏，和创建一个线程安全的 Array 和 Dictionary](https://poos.github.io/2021/03/07/SafeArrayDic/)


#### 测试Xcode 12 + 按钮 title 没有被成功设置的问题

**TestOCButtonExtension**

因为 setTitle(:) 与系统的冲突。

[Swift 的 Array 和 Dictionary 源码欣赏，和创建一个线程安全的 Array 和 Dictionary](https://poos.github.io/2021/03/07/SafeArrayDic/)



#### 测试循环引用

**TestRetainCycle**

属性持有（如果没有相互），不一定循环引用。

如果需要持有，考虑使用weak ，unowned。


#### Stroyboard Table + Cell

**TestStroyBTable**

在 Stroyboard 可以直观看到结果。

免去 注册 table view cell，直接取用。


#### 非常简单的 Swift UI 测试

**TestSwiftUI**

测试简单的Label和list显示。


#### 不正确使用 UIAlertController 会造成内存问题

**TestSystemAlert**
**TestWeakVarVC**

需要使用 `[weak a]` 打破循环。

```
class A: UIAlertController {
    deinit {
        print("🧩--\("")\n--🧩--\(Date())--\(#function)--\(#line)🧩")
    }
}

class ViewController: UIViewController {
    @objc func tapA() {

        let a = A(title: "title", message: "message", preferredStyle: .alert)
        a.addAction(UIAlertAction(title: "button title", style: .default, handler: { [weak a] (action) in
            a?.dismiss(animated: true, completion: nil)
        }))
        self.present(a, animated: true, completion: nil)
    }

}
```


#### 使用Dispatch制作Timer

**TestTimer**

如题， 使用Dispatch制作Timer，仅供参考。


#### UserDefaultKey 写个100+会不会造成key越界

**TestUserDefaultKeyMaxLength**

> UserDefault 快速存取需要同步。但是key过大不会有问题。

不会的，kv共享约几M的数据，并不会有问题。

随机码加UserId 存储唯一信息是可以的，数据结果不会出错。



#### 测试在alert的情况下弹窗出新的VC

**TestW**

使用general的方式取出当前的keywindow，使用keywindow的 rootvc  preset 出新的vc。


骚操作，非急勿用。



#### WKWebview 在 Storyboard使用

**TestWKWebViewOnInterfaceBuilder**

使用 View 在Xib；指定 Custom Class；Code 上继承 WKWebView 即可



### 完～
