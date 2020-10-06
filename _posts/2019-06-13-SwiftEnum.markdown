---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 中 高级枚举使用，模式匹配                 # 标题
subtitle:   Swift GG 关于模式匹配的资料，实践枚举和模式匹配的开发             #副标题
date:       2019-06-13                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
- Code
---

### 相关链接

#### [https://swift.org/](https://swift.org/)
##### [Swift 中枚举高级用法及实践](https://swift.gg/2015/11/20/advanced-practical-enum-examples/)

swift 中的枚举有很强大的功能：

- 可以指定枚举类型，int，string，甚至是 class 类型（需要通过 String 遵守协议转换）

- 枚举可以嵌套使用，这在定义宏往往很有用

- 枚举可以关联值，也就是可以带参数，所以有广泛使用的 [github.com/Moya](https://github.com/Moya/Moya) 库的出现

- 枚举可以添加方法和属性，包含静态方法，可变方法（能够改变self）

- 因为枚举可以添加方法和属性，所有枚举甚至可以遵守协议，这又给了枚举以无限可能

-  使用字符串赋值枚举等神奇操作

Swift 中枚举被用作网络框架，统计打点框架，Router框架，Theme框架等等。当然除了框架以外，用在一些业务也是非常贴切，本文就是一个简单的栗子～


##### [Swift Pattern Matching详细信息](http://appventure.me/2015/08/20/swift-pattern-matching-in-detail/)

swift 中的模式匹配，switch case：

- 与枚举配合使用，方便区分。特别是关联了值的枚举，会非常有用

- 可以使用元组，通配符 '_'，所以在匹配时候会更强大

- 配合 swift 的表达式，可以实现判断 self 或者筛选 case 的功能

[详解 Swift 模式匹配](https://swift.gg/2015/10/27/swift-pattern-matching-in-detail/)

### 使用范围

因为 swift 的这些特性，enum 的使用范围会非常广：


**带参枚举，方法属性，可选值，类型匹配，范围匹配，遍历匹配，Api 规法化，宏定义，类簇，模型解析...**


### 例子

下边是一个真实的例子，需求是展现一个个页面情景，根据不同的选择，产出最终产出结果，有点像情景交互游戏，或者年度账单之类：

**利用枚举实现类簇，保证项目风格，且容易调试**：

CView定义了当前页面的所有元素，背景色或view，前景view，焦点，可选按钮等...

```swift
enum CView {
    case back
    case black
    case eye(position: CGPoint)
    case ground(image: String)
    case button(position: CGPoint, size: CGSize)

    var node: UIView {
        switch self {
        case .back:
            let back = UIView.init(color: UIColor.white)
            //...
            return back
        case .ground(let image):
            //...
        case .button(let position, let size):
            //...
        case .black:
            //...
        case .eye(let position):
            //...
        }
    }
}
```


**利用枚举定义关卡，利用方法返回关卡的背景音乐，同时返回需要添加的view，和 view 对应的事件（使用元组的方式）**：

如果说第一个枚举只是当前场景的所有view列举的话，第二个就是枚举场景并且提供场景转换...

```swift
enum Level {
    case welocme
    case start
    case choose
    case levelSelf
    case pay
    case copyright


    //初始化界面，返回
    var music: String? {
        switch self {
        case .start, .levelSelf:
            return "start"
        case .pay:
            return "pay"
        default:
            return nil
        }
    }

    //初始化界面，返回
    func nodes() -> ([UIView], [(UIView, Level)]) {
        switch self {
        case .name:
            let ground = CView.ground(image: "name").node
            let button0 = CView.button(position: CGPoint(x: 0, y: -70),
                                      size: CGSize.init(width: 450, height: 100)).node
            return ([ground, button0], [(button0, .changeName)])

            //...
        default:
            break
        }
        return ([], [])

    }
}
```


可以看到通过这个几个枚举的创建，一个个页面的选择和选择之后的结果都十分明显的确定了。很明显在调试应用的时候能够避免很多的错误。


写完框架之后，需要的只是添加和修改 enum 这一块了，这种方式在情景对话中非常适合。当然可以更加具体的扩展枚举提供更多的功能。


### 总结

枚举在 swift 中得到了重大的提升，如果你使用 swift 开发，一定不要错过这个特性。相信掌握了这个技能之后，你的 level 更上一步。
