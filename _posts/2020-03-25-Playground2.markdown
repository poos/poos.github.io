---
layout:     post
title:      那些年写过的 Demo +～
subtitle:   内存测试, DispatchSemaphore, Enum, String
date:       2020-03-25
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Code
- Demo
---

### 前

列一下写过的 Demo，总结和备查

纯干货篇～

上一篇是相似的内容！

[Playground](https://poos.github.io/2020/03/21/Playground/)


Demo 网址：

[poos/SomeDemo](https://gitee.com/poos/SomeDemo)


#### 内存测试

**M_Cycle**

测试循环引用


#### DispatchSemaphore 同步线程

**M_DispatchSemaphore**


使用信号量，等待多个请求完成

```

        let semaphore = DispatchSemaphore(value: 0)

        print("🧩🧩--\("login")--🧩--\(Date())--\(#function)--\(#line)🧩")

        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // api call complete
            self.userID = "user"
            semaphore.signal()
        }

        semaphore.wait()
        print("🧩🧩--\("login success")--🧩--\(Date())--\(#function)--\(#line)🧩")

        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // api call complete
            self.amount = 100
            semaphore.signal()
        }
        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // api call complete
            self.address = "Xxx xx Ste 100"
            semaphore.signal()
        }
        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // api call complete
            self.cardNumber = "1000 1000 1000 1000"
            semaphore.signal()
        }
        semaphore.wait()
        semaphore.wait()
        semaphore.wait()

        print("🧩🧩--\("success")--🧩--\(Date())--\(#function)--\(#line)🧩")
        print(self.userID)
        print(self.amount)
        print(self.address)
        print(self.cardNumber)
```



#### Enum

**M_Enum**


- 带参数的枚举，equatable 结果如何
- 使用 allCase（CaseIterable），优化init


```
var str = "Hello, playground"

  enum VCIndex: Int, CaseIterable {
      case home
      case share
      case setting = 5
      case none = -1

      init(t: Int) {
          if let index = VCIndex(rawValue: t) {
              self = index
          } else {
              self = .none
          }
      }

  }

  print(VCIndex(t: 3))

  VCIndex.allCases.forEach { (item) in
      print(item.rawValue)
  }

```


#### UILabel 的 Linebreak 为什么第二行会两个单词。

**M_LabelLinebreak**

苹果似乎不喜欢只有两行的时候第二行显示单独的一个单词！（至少显示两个）

解决方法在 string 后边 添加四个空格 `    `。 非官方方法，非常神奇的问题，和非常神奇的解决方法。



#### Operation 同步线程

**M_DispatchSemaphore**


使用Operation，等待多个请求完成。


#### String

**M_String**


- OCR 相似字体替换
- String Nil isEmpty
- 使用 spellOut， `1` -> `one`， `15` -> `fifteen`

```
    var str = "Hello, playground"

    let number:NSNumber = 15
    let formatter = NumberFormatter()
    formatter.numberStyle = .spellOut
    let numberString = formatter.string(from: number)
```
- 字符串处理 `0.171254 0.190054 0.225327 1` -> `B7BFCA`


#### Unowned 应该怎么用

**M_UnownedSelf**

```

var str = "Hello, playground"

class A {
    let name = "A"

    lazy var attributeString: NSAttributedString? = { [unowned self] in
        return NSAttributedString(string: self.name)
    }()

}


print("🧩🧩--\(A().attributeString?.string)--🧩--\(Date())--\(#function)--\(#line)🧩")

```

### 完～
