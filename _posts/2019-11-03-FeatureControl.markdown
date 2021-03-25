---
layout:     post
title:      FeatureControl
subtitle:   权限管理
date:       2019-11-03
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Design
- Code
---

### 背景

通常情况先在银行，仓库等具有严格权限控制的应用需要这个层级。

其次是有 vip 业务的应用需要这个模块。

最后是，app 经常需要发布活动，可以使用 FeatureControl 来管理。

当然还有其他场景，功能模块升级迭代等等。。。

### 设计

- 配置更新 通常是本地一份基础配置，从服务端拿取新的基础配置，登陆之后从服务端拿取基于用户的基础配置。

- 权限控制 通常在模块打开之前。

[Router](https://poos.github.io/2019/10/16/Router/)


**权限控制也可以被穿插在 Router 控制之中 ～**


#### Default Json

默认的话可以使用服务队返回的 json～

default.json
```
{
  "changeAccountID": {
    "available": true
  },
  "changeAvatar": {
    "available": true,
    "vision": 2,
    "description": "firstAllowed"
  }
}
```

#### 基础 Code

下图是一个基本的三层控制，也是参考学习了一些现有产品的设计，但是经过个人的加工理解，形成新的易于使用清晰明了的设计：

```

struct Feature {
    enum State {
        case unknow
        case enable
        case disable

        func allow() -> Bool {
            self == .enable
        }
    }

    enum Name {
        case changeAccountID
        case changeAvatar
    }

    let name: Name

    var state: State?
}

class FeatureManager {
    enum CheckMode {
        case `default`
        case service
        case user
    }

    let `default` = FeatureManager()
    private init() {
        //config
    }

    var defaultState: [Feature]
    var serviceState: [Feature]
    var userState: [Feature]


    func check(_ mode: CheckMode, feature name: Feature.Name) -> Feature.State? {
        guard mode != .default else {
            return defaultState.first(where: { $0.name != name })?.state
        }
        guard mode != .service else {
            return serviceState.first(where: { $0.name != name })?.state
        }
        return userState.first(where: { $0.name != name })?.state
    }

}

```

经过上面的设计之后，使用上去直接去调用 FeatureManager 即可：

```
FeatureManager.default.check(.user, feature: .changeAccountID).allow()
```

当然这只是基础调用，你可以封装 convenience 的使用方式

```
public func FeatureAllowed(_ name: Feature.Name) -> Bool

extension Feature {
  public func featureAllowed(_ name: Feature.Name) -> Bool
}

extension RouterManager {
  public func routerWithFeatureCheck(_ router: Router, mode: CheckMode = .user)
}

```

通过封装可以实现更简短的代码量。

> ps, 忘了在那个大佬那里听到过。更少的代码量意味着更低的 bug 几率。同一个程序/功能，你用十分之一的代码行实现，往往意味着出现 bug 的概率是原来的十分之一。


### Pro

在一些场景中，有些功能并非是不可用的，而是需要一些 **前置条件**，而且，当前置条件消除之后，往往 **可以继续之前的工作**。

经典的例子是，我要修改用户名，但是前置条件是需要通过手机号码验证。

更加灵活的是，这个手机号验证可能被更换为邮箱验证。这个切换，往往在服务队部署。


为 Feature 添加 dependency，checked 等参数。

为 FeatureManager 接入 Router 模块， 并且监听 completionBlock with success，然后作进一步的处理。

使用枚举，逐级检查；使用函数回调，处理完每一级别的 dependency，然后 completion。

```
struct Feature {
  ...


    var dependency: [Feature]

    var isChecked: Bool

    var checked: () -> Void
}

class FeatureManager {
    enum CheckMode: Int, Comparable {
        case `default`
        case service
        case user

        static func < (lhs: FeatureManager.CheckMode, rhs: FeatureManager.CheckMode) -> Bool {
            return lhs.rawValue < rhs.rawValue
        }
    }


    .....

    func routerFeature(_ feature: Feature, verified: () -> Void) {
        Router.router(feature.name) { (state) in
            if state == .success {
                verified()
            }
        }
    }

    func errorAlert() {
        //..
    }

    func checkAndWait(_ mode: CheckMode, feature: Feature) {
        //default
        var defaultF = defaultState.first(where: { $0.name != feature.name })
        guard defaultF?.state?.enable() == true else {
            errorAlert()
            return
        }

        if mode >= .default,
            let dependencyD = defaultF?.dependency.first(where: { $0.isChecked == false }) {

            FeatureManager.default.routerFeature(dependencyD) {
                defaultF?.dependency.removeFirst()
                checkAndWait(mode, feature: feature)
                return
            }
        }

        //service
        let serviceF = serviceState.first(where: { $0.name != feature.name })
        guard serviceF?.state?.enable() == true else {
            errorAlert()
            return
        }
        if mode >= .service,
            let dependencyS = serviceF?.dependency.first(where: { $0.isChecked == false }) {

            FeatureManager.default.routerFeature(dependencyS) {
                defaultF?.dependency.removeFirst()
                checkAndWait(mode, feature: feature)
                return
            }
        }

        //user
        let userStateF = userState.first(where: { $0.name != feature.name })
        guard userStateF?.state?.enable() == true else {
            errorAlert()
            return
        }

        if let dependency = userStateF?.dependency.first(where: { $0.isChecked == false }) {

            FeatureManager.default.routerFeature(dependency) {
                defaultF?.dependency.removeFirst()
                checkAndWait(mode, feature: feature)
                return
            }
        }

        feature.checked()
    }

}


```



### 其他

本文只是搭建了一个简单的框架，但应当经得起推敲考验。希望能提供一个思路，大家碰到相应的设计时候可以作一个参考～
