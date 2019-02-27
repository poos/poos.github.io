---
layout:     post                       # 使用的布局（不需要改）
title:      华容道搜索算法实现             # 标题
subtitle:   通用协议，路径查询，多叉树节点，比特位的使用，优化            #副标题
date:       2019-01-20                # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 算法
---

> HuaRongDao

 **项目地址：[ShownBie/HuaRongDaoSwift](https://gitee.com/poos/HuaRongDao)**

本文介绍华容道的搜索算法的构建和实现。

### 协议

使用协议可以让代码看起来更调理，更加有框架。而且可以将框架部分简单的应用于其他搜索搜索游戏。


通过制定通用的搜索协议，可以将搜索的算法规范化：某个状态（包含父节点），状态权重，搜索器

```
//某个状态
protocol SXNodeState {
    var parentState: SXNodeState? { get set }

    var data: [Int] { get set }

    var identifier: Int64 { get set }

    func childeStates() -> [SXNodeState]
}

//某个状态的权重 暂未使用
protocol SXAStarState {
    var fromValue: Int { get set }

    var toValue: Int { get set }

    var totalValue: Int { get set }

    func toTargetStatus(_ from: SXNodeState) -> Int
}

//搜索器
protocol SXSeacher {
    var startState: SXNodeState { get set }

    var targetState: SXNodeState { get set }

    var successComparator: (SXNodeState, SXNodeState) -> Bool { get set }

    func search() -> Bool

    func pathWithState() -> [SXNodeState]
}
```

### 多叉树

搜索算法通常以一种 **多叉树** 的形式存在，树的每一个节点如下：**父** 节点，**孩子们** 节点，当前的 **值**， **唯一标识**（用于查询当前节点是否被查询过）

```
protocol SXNodeState {
    var parentState: SXNodeState? { get set }

    var data: [Int] { get set }

    var identifier: Int64 { get set }

    func childeStates() -> [SXNodeState]
}

```

### 权重

```
protocol SXAStarState {
    var fromValue: Int { get set }

    var toValue: Int { get set }

    var totalValue: Int { get set }

    func toTargetStatus(_ from: SXNodeState) -> Int
}
```

A* 算法的必备要素，根据不同的规则大概可以分为几个值：**到开始的权重**，**到成功状态的权重**，**总权重**。需要传入开始点，结束点和当前点

### 搜索类

搜索类继承 **搜索协议**， 然后添加几个私有变量用于计算存储即可。

如下添加了属性 pathStates，historyIdentifier，succcesState 和 方法 getChild()


```
class HuaSearch: SXSeacher {
    var startState: SXNodeState

    var targetState: SXNodeState

    var successComparator: (SXNodeState, SXNodeState) -> Bool

    init(start: SXNodeState, target: SXNodeState) {
        //...
    }

    func search() -> Bool {
        //...
    }

    func pathWithState() -> [SXNodeState] {
        // ...
    }

    //MARK: private

    private var pathStates = [SXNodeState]()

    private var historyIdentifier = Set<String>()

    private var succcesState: SXNodeState?

    private func getChild(state: SXNodeState) -> [SXNodeState] {
        let child = state.childeStates()
        return child
    }

}
```


### 关于比特位

当状态可以使用一个 **64位Int 型** 表示的时候，可以考虑使用二进制位来记录当前 Identifier，这会大幅提升算法速度。

也可以考虑 **直接使用 Int64 同时作为 data 和 Identifier**

那么关于二进制位的运算就要了解一下了：

```
extension Int64 {
    // 取空格位置
    func space1Index() -> Int64 {
        return self & 0b11111
    }
    func space0Index() -> Int64 {
        return self >> 5 & 0b11111
    }
    // 取某位的值
    func valueAtByte(index: Int64) -> Int64 {
        return (self >> 10) >> (40 - 2 - 2 * index) & 0b11
    }
    // 根据一个 identifier 推出另一个 identifier
    func mapToAnotherIdentifier() -> Int64 {
        func exchange(value: Int64, length: Int64 = 8) -> Int64 {
            var left = value >> 4
            var right = value & 0b1111
            left = (value >> 2) | (value << 2)
            right = (right >> 2) | (right << 2)
            let result = (left << 4) | right
            return result
        }
        let base = self >> 10

        var first = base >> 16
        var second = base >> 12 & 0b11111111
        var third = base >> 8 & 0b11111111
        var fourth = base >> 4 & 0b11111111
        var fifth = base & 0b11111111

        first = exchange(value: first)
        second = exchange(value: second)
        third = exchange(value: third)
        fourth = exchange(value: fourth)
        fifth = exchange(value: fifth)

        var result = (first << 16) | (second << 12) | (third << 8) | (fourth << 4) | fifth

        let space0 = space0Index()
        let space1 = space1Index()
        result &<<= 5
        result += Int64(Int64(space0 / 4) * 4 + (3 - space0 % 4))
        result &<<= 5
        result += Int64(Int64(space1 / 4) * 4 + (3 - space1 % 4))

        return result
    }
    // 用于交换某两个比特位的值
    mutating func exchangeByte(from: Int64, to: Int64, isSpace0: Bool = true) {
        func exchange(value: Int64, length: Int64, from: Int64, to: Int64) -> Int64 {
            let value0 = value >> (length - 2 * from) & 0b11
            let value1 = value >> (length - 2 * to) & 0b11
            let set0 = value & ~(0b11 << ((length - 2 * from))) & ~(0b11 << ((length - 2 * to)))
            let valueNew0 = value1 << (length - 2 * from)
            let valueNew1 = value0 << (length - 2 * to)
            let result = set0 + valueNew0 + valueNew1
            return result
        }

        let exchangePrefix = exchange(value: self >> 10, length: 40, from: from + 1, to: to + 1)

        let suffix: Int64
        if isSpace0 {
            suffix = self & 0b00000000000000000000000000000000000000001111100000 + to
        } else {
            suffix = self & 0b00000000000000000000000000000000000000000000011111 + to << 5
        }
        self = exchangePrefix << 10 + suffix
    }
}

```

### 优化


项目分3个分支，都是使用了广度优先算法进行遍历计算，区别在于用于存储的节点使用的数据类型不一样

- **master： 采用字符串作为标识记录和存储节点**

```
protocol SXNodeState {
    var parentState: SXNodeState? { get set }

    var data: [String] { get set }

    var identifier: String { get set }

    func childeStates() -> [SXNodeState]
}
```

- **Preview： 使用[Int]记录数据，使用 Int64 记录状态，区分重复**

```
protocol SXNodeState {
    var parentState: SXNodeState? { get set }

    var data: [Int] { get set }

    var identifier: Int64 { get set }

    func childeStates() -> [SXNodeState]
}
```

- **Ultimate： 只使用了 Int64 记录当前状态，并且作为唯一标识**

```
protocol SXNodeState {
    var parentState: SXNodeState? { get set }

    var identifier: Int64 { get set }

    func childeStates() -> [SXNodeState]
}
```






#### 关于 Ultimate

通过 0（曹操） 1（横将） 2（纵将） 3（单个）来使用二进制位记录当前状态，在分别使用两个5位的数据用来表示两个空格的位置

例如如下布局的表示为：

```
卒卒张赵
关关张赵
曹曹马马
曹曹黄黄
卒卒〇〇

-------十进制如下----------
3322
1122
0011
0011
3333

其空格分别为18，19
--------二进制即是---------
11 11 10 10
01 01 10 10
00 00 01 01
00 00 01 01
11 11 11 11

其空格分别为10010，10011
--------所以其最终 id 为 50 位的二进制数---------

let identifier: Int64 = 0b11111010010110100000010100000101111111111001010011

```

#### 关于效率

1. 无疑最慢
2. 使用 Set<Int64> 记录历史之后快了很多，得益于 **swift 数组的性能**，运算速度比较快快的
3. identifier 即数据，内存占用大大优化；虽然位运算速度快很多，但是大量的位移和判断对性能有影响

#### 更进一步的优化

**这部分不准备在这个项目继续更新了**


1. **增加权重**

鉴于华容道的特殊性，其最终状态是不确定的（只要曹操走到下部最中间即可），所以计算接近于最终状态的事情变得复杂了


**如果非要使用也不是不可以** ： 华容道现在有 400 多种初始布局，对应的有400多种最终布局，分别记录并增加权重即可。


本项目只是研究了一种布局，理论上可以添加一个最终布局，增加权重


**还有一个问题，华容道有很多类似重复的来回走动，为了将一些子交换位置，所以加权的效果是有限的**



2. **最快解法**

**上一条说过** ：华容道有有限的布局

**并且华容道是有 一横，二横，三横，四横等不同的解法步骤**

所以如同魔方一样，一定有一些有限的中间布局，只需要从初始转换为中间布局，即可根据既定步骤求出解。


3. **结合资料和算法**

广度优先求解 15 步以内在 0.5 秒以内，如果再加上加权，0.5秒能解出更多的步骤。

最简单的方式应该是下边的样子

**利用一个个关键点（木桩）的记录，使用搜索（跳）求解**


### 总结

**华容道** 比普通的 **九宫格拼图** 求解更加复杂。因为不确定最终状态，所以A*没法发挥威力；因为存在多种初始布局，且对应多种最终布局，所以让普通搜索求解无能为力。但是（下边换一段）...

**像魔方算法一样，虽然中间过程非常复杂，但是通过有限的中间状态转换，依然可以在秒级算出正确解法**。
