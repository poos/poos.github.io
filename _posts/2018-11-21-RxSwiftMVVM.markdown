---
layout:     post                       # 使用的布局（不需要改）
title:      RxSwift 的 MVVM 框架理解使用              # 标题
subtitle:   适用于 RxSwift 的 各个框架， 常用框架的分析            #副标题
date:       2018-11-20                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 优化
---

### 设计模式

先来理解一下移动端，前端流行的设计模式：MVC，MVP，MVVM

MVC： 苹果经典的模式，瘦的 view，瘦的 model，臃肿的 Controller；C 控制事件，更新 model，协调从 model 更新至 view

MVP：MVC 的改良模式，P 作为 V 和 M 的协调中心，交互发生在 P 内

MVVM：MVP 的更进一步版本，view 和 model 通过绑定进行通信和同步，事件通过绑定控制 UI


**总结一下**

1. 简单的页面使用 MVC 即可

2. 对于复杂的页面，可以考虑使用 MVP / MVVM 的模式，**注意 mvvm 通常是引入了 Reactive 框架以实现绑定**

3. 除了这两种方式，也有其他易于理解的清晰的，可支持复杂页面的设计模式

#### input - output

通过定义页面输入和输出来规范 VM / P 部分。

 例子中采用的是 RxSwift + MVVM 的 input-output 设计。实际上，不一定要使用 Reactive 框架，通过输入输出的约束，也可以方便测试。只不过需要处理的事件可能会多一倍（因为有些地方是需要双向绑定的）。

```
public protocol ButtonViewModelInputs {
    var taps: PublishSubject<Void>{ get }
}

public protocol ButtonViewModelOutputs {
    var result: Driver<Bool> { get }
    var isLoading: Driver<Bool> { get }
}

public protocol ButtonViewModelType {
    var inputs: ButtonViewModelInputs { get  }
    var outputs: ButtonViewModelOutputs { get }
}

final class ButtonViewModel: ButtonViewModelType, ButtonViewModelInputs, ButtonViewModelOutputs {

}

```

可以看出，通过以上的方法，规范了输入输入，就使得复杂页面可以测试，可以拆分事件。

#### Reactor

路由模式，通过 事件 -> 中间操作（private）-> 状态。

可以看出这个同样是基于 RxSwift + MVVM 的，但是这个模式同样适用于非 Rx 的模式。通过事件，处理事件，产生最终的页面状态（不使用 rx 就需要在最后一步自己读取 VM / P 的状态进行加载）。

```
final class TextReactor: Reactor {

    enum Action {
        case request
    }

    enum Mutation {
        case reset
        case setLoading(Bool)
    }

    struct State {
        var model: TestModel
        var times: Int
        var isLoading: Bool
    }

    let initialState: TextReactor.State

        init() {
            let initModel = TestModel.init(name: "none", age: 0)
            self.initialState = State(model: initModel, times: 0, isLoading: false)
        }

        // MARK: actions -> Mitation
        func mutate(action: TextReactor.Action) -> Observable<TextReactor.Mutation> {
            let request = Observable.just(Mutation.reset)
                .delay(1, scheduler: MainScheduler.instance)

            return Observable.concat([
                Observable.just(Mutation.setLoading(true)),
                request,
                Observable.just(Mutation.setLoading(false))
                ])
        }

        func reduce(state: TextReactor.State, mutation: TextReactor.Mutation) -> TextReactor.State {
            var state = state

            switch mutation {
            case .reset:
                state.model = TestModel.test()
            case .setLoading(let isLoading):
                state.isLoading = isLoading
            }
            return state
        }
```

### 设计模式之外

再优秀的框架如果滥用，都会产生各种问题。如果代码不加一约束，随意书写还是会造成页面巨大，难以维护。在框架之外的一些开发经验和新的分享一下


**针对比较复杂的页面，进行的代码约束(基于 MVVM)**

#### 1. Model 部分

model 部分最多可以分为3部分，一般分为2部分。

- 第一部分为基础 model 数据，用于存储元数据 （通常对应于服务端返回的 json 对象，项目中可以重复复用）

- 第二部分为含显示数据的数据的，继承于第一个 model，用于扩显示的元素，例如 URL，Attributestring，（table 等列表建议使用 存储属性 优先性能，其他可以使用计算属性，优先内存），虽然如此但不应该引用 View

- 第三部分为组合生成页面对应的 model， 常用于多个接口组合同一组件下的 model

对 model 部分充分的赋能，让它掌握更多的代码，其他模块就能得到更优的代码，这也是为整个框架做出了杰出的贡献了。详细部分可以看本章小结部分 **[关于 tableview 的优化]** ，充分释放 model 权利的魅力也在其中展现淋漓尽致。


#### 2. View 部分

- 创建某个页面的类簇，通过类簇生成页面所需的 view

- view 创建处于一个类，且应当不包含 model 信息

view 部分的重要性在于封装极限的封装小的组件，这样整个项目风格统一，能够避免 copy代码情况出现。小结部分 **[关于 tableview 的优化]** 也有介绍。

#### 3. VM 部分

- 可以使用 Reactive 框架，也可以不使用。但是推荐使用 input - output 或者 Reactor 来进行约束，确保代码清晰

- 在 vm 部分创建 model 和 view 的分类（独立于 VM 维护，当然也可以创建另外一个页面），用于提供便利的 model <==> view 更新赋值

VM 部分代码是一个组件的核心，在保证内容正确的情况下可以通过泛型逐步拆分复用，这也是项目优化的点。大 VM 套用 基础 VM。

#### 小总结

设计模式的出现就是因为代码分配不均，代码耦合，阅读困难，难以测试。那么就 **需要一些约束去规范我们的代码**，不管是 MVC， MVP ，MVVM 还是一些特意设计的其他规则，其目的就是为了使得代码易于理解维护。当然，**引入越多的第三方，就会使得起步更加难，而带来的好处就是代码更优**。

**抛开业务谈设计模式就是扯淡**，所以大家还是应该根据对应模块的复杂度来设计。

上边是经历了一个复杂的组件设计之后得出的一些经验，希望能帮助到大家，这应该是第二篇，第一篇是 **[关于 tableview 的优化](https://poos.github.io/2018/11/20/TableView/)**


### 适用于 RxSwift 的 各个框架

官方文档为我们推荐了三种框架


- MVVM - 当今非常流行的 MVVM 设计模式

- ReactorKit - 结合了 Flux 和响应式编程的架构

- RxFeedback - 由 RxSwift 创始人（Krunoslav Zaher） 提供的一个反馈循环架构

在 github 上更受欢迎的前两个已经在前文中提到过，第三种模拟一个反馈循环系统：

```
事件 -> 系统状态 -> 请求 -> 状态变化（更新 ui 等
```
这个号称 RxSwift 最简单的架构，但是简单就意味着局限性，我在一步步绑定的过程中完成了整个操作。但是要进行一些高级操作（flatMapFirst）时候碰到了问题（有人有好的方式也可以分享下）。不过我更愿意使用相对开放 MVVM 架构。通过协议约定输入输出，黑盒维护和测试是很方便的。

### 总结

首先说一下，本文用到的 架构/框架 一词，其实意思是一个业务组件的设计，本类我是更愿意用设计这个词，无奈很多中文文档将其称之为架构，所以为了大家理解我是用了框架这个词语更系统架构进行区分。

组件内的相关类都是耦合严密的，通过一些约束进行规范化，使得代码更好，这就是代码架构的过程。这些约束就是上边讨论的框架。最后在强调一下：**不要抛开业务谈设计，设计应该跟业务是密切相关的。**




> ----

[如果想了解其他相关的内容](https://poos.github.io/tags/)
