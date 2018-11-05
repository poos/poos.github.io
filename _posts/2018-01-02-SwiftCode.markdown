---
layout:     post                       # 使用的布局（不需要改）
title:      Swift代码妙用                 # 标题
subtitle:   方法嵌套妙用、泛型妙用等           #副标题
date:       2018-01-01                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: false                         # 是否归档
tags:                                #标签
- Code
---


### to be continue ...

### 方法嵌套妙用

#### 处理循环引用

<!--
如下的代码发生了循环引用：


```
class A: NSObject {

    let block: (Bool) -> Void

    init(block: @escaping (Bool) -> Void) {
        self.block = block
    }

    deinit {
        print("\(self)A deinit")
    }

}
```

```
class B: NSObject {
    var a = false
    var aC = A.init { (_) in

    }

    func testBlcok() {

      //self 持有了 aC，aC 的 block 持有了 self.a
        aC = A.init(block: { (isT) in
            self.a = isT
        })
        aC.block(true)
        print(aC)
    }

    deinit {
        print("\(self)😆B deinit")
    }
}
```

当在 VC 中调用时候就会造成对象无法被 deinit

**不要在 playground测试内存泄露相关的代码**, [stackoverflow: Deinit method is never called - Swift playground](https://stackoverflow.com/questions/24363384/deinit-method-is-never-called-swift-playground)

```
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.

        var b = B()
        b.testBlcok()
        let c = B()
        c.testBlcok()
        b = c
    }     


}
```

如果添加 **[weak self]** 即可防止强引用，本文探讨的是另外一种方式：函数嵌套。修改 testBlcok 如下：

```
...
func testBlcok() {
    func blockAction(isT: Bool) {
        a = isT
        print("hahhah\(isT)")
    }

    aC = A.init(block: { (isT) in
        blockAction(isT: isT)
    })
    aC.block(true)
    print(aC)
}
...

```

**此时就可以脱离循环引用正常释放了**


##### 为什么会这样呢

因为作用域的问题

[Swift 学习笔记4 —— 文件结构，作用域和生命周期](https://n3xtchen.github.io/n3xtchen/swift/2016/02/14/swift-tut4)

上边代码中被嵌套的部分的作用域只在 { 开始和 } 结束之间，所以
```
func blockAction(isT: Bool) {
    a = isT
    print("hahhah\(isT)")
}
``` -->
