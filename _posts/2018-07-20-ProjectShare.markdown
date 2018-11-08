---
layout:     post                       # 使用的布局（不需要改）
title:      Swift项目多处调用分享的设计方案                 # 标题
subtitle:   使用协议清晰明了的设计分享              #副标题
date:       2018-07-20                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 设计
---

> 当一个页面包含了太多的太杂的内容页面就变得复杂了，把相似的内容抽出来单独管理，让项目更清晰。


## 分享调用的设计方案 Protocol-Class

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
//可能是 View
//可能是 WebView
//可以是 任何类
class A {
    func xx {
        share()//调用协议方法
    }
    func jsShare() {
        share()//调用协议方法
    }
    func jsShareMoment() {
        shareMoment()//调用协议方法
    }
}
```

这里是协议分发管理的 protocol类, 在这里实现分享的方法
```
// 定义统一的协议
extension A: ShareProtocol, ShareProtocol2 {
    func share() {
        //...
    }
    func shareMoment() {
        //...
    }
}

```


### 总结

通过这种设计，ShareManager更加彻底的负责所有的分享事宜；VC 只是简单调用，页面结构更简单了。


---
>
---

## 分享功能实现的框架


## 分享框架

分享的具体实现就不必细说了


File | Description
---|---
Model | 分享模型
View | 分享视图
ViewControl | 处理弹出动画; 分流选定后的操作
Manager | 封装各个SDK分享方法



>   
    func present() {
    let model = ShareModel()
    ...
        AppShareViewControl.present(model)
}

>   



## Model

> 略去


## View
> skip



## ViewControl

view 的一些动画，大部分简单的动画就不再介绍。

**设置按顺序弹出动画,layout位移动画**
```   


    static private func buttonsAnimation(buttons: [UIView]) {
        for (index, button) in buttons.enumerated() {
            button.layer.setValue(375, forKeyPath: "transform.translation.y")
            button.alpha = 0
            UIView.animate(withDuration: 0.5,
                           delay: 0.05 * Double(index) + 0.1,
                           usingSpringWithDamping: 0.7,
                           initialSpringVelocity: 0.7,
                           options: .curveEaseInOut,
                           animations: {
                           button.alpha = 1
                           button.layer.setValue(0, forKeyPath: "transform.translation.y")
                           })
    }

```   

## Manager

#### 调用系统分享
```
    // MARK: - 系统
    //系统分享传入[String, UIImage, URL]
    //注意:系统分享qq会调用URL 对应的web更新数据,若没有对应web页（非 H5页）会显示异常
    func shareSystem(title: String, content: String?, image: UIImage?, link: String) {
        var items: [Any] = []
        if title.length > 256 {
            items.append((title as NSString).substring(to: 255))
        } else {
            items.append(title)
        }
        var thumbimage = image
        if image == nil {
            thumbimage = UIImage.init(named: "AppIconSmall")
        }
        if thumbimage!.size.height > 70 {
            thumbimage = imageCompressSize(image: thumbimage!, size: CGSize.init(width: 70, height: 70))
        }
        items.append(thumbimage! as Any)
        items.append(URL.init(string: link)!)
        let activity = UIActivityViewController.init(activityItems: items, applicationActivities: nil)
        UIViewController.currentViewController().presentVC(activity)
}
```    


#### 调用微信分享(建议优化使用 model)
```

    fileprivate func share(type: Int, title: String, content: String?, image: UIImage?, url: String) {
        if !WXApi.isWXAppInstalled() {
            let popview = PopupView()
            popview.popupText(.noWechat)
            return
        }

        let message = WXMediaMessage.init()
        if title.length > 256 {
            message.title = (title as NSString).substring(to: 255)
        } else {
            message.title = title
        }
        if let description = content {
            if description.length > 512 {
                message.description = (description as NSString).substring(to: 511)
            } else {
                message.description = description
            }
        }
        var thumbimage = image
        if image == nil {
            thumbimage = UIImage.init(named: "AppIconSmall")
        }
        if thumbimage!.size.height > 70 {
            thumbimage = imageCompressSize(image: thumbimage!, size: CGSize.init(width: 70, height: 70))
        }
        if let data: Data = imageCompressData(image: thumbimage!, max: 32) {
            message.thumbData = data
        }

        let webpage = WXWebpageObject.init()
        webpage.webpageUrl = url
        message.mediaObject = webpage

        let send = SendMessageToWXReq.init()
        send.bText = false
        send.message = message
        let scene = type == 0 ? WXSceneSession : WXSceneTimeline
        send.scene = Int32(scene.rawValue)

        WXApi.send(send)
    }

    fileprivate func share(image: UIImage) {
        if !WXApi.isWXAppInstalled() {
            let popview = PopupView()
            popview.popupText(.noWechat)
            return
        }

        let message = WXMediaMessage.init()

        // image最大10M
        let wxImage = WXImageObject.init()
        if let data: Data = imageCompressData(image: image, max: 10 * 1024) {
        wxImage.imageData = data
        }
        message.mediaObject = wxImage

        let send = SendMessageToWXReq.init()
        send.bText = false
        send.message = message
        send.scene = Int32(WXSceneSession.rawValue)

        WXApi.send(send)
}

```    


#### 压缩图片等方法
```
    // MARK: - 通用
    fileprivate func imageCompressSize(image: UIImage, size: CGSize) -> UIImage {
        let result = image
        UIGraphicsBeginImageContext(size)
        result.draw(in: CGRect.init(x: 0, y: 0, width: size.width, height: size.height))
        UIGraphicsEndImageContext()
        return result
    }

    fileprivate func imageCompressData(image: UIImage, max: Int) -> Data? {
        var strength: CGFloat = 0.99
        var data = UIImageJPEGRepresentation(image, strength)
        if data == nil {
            return nil
        }
        while data!.count > max * 1024 {
            if strength < 0.4 {
                let image = UIImage.init(named: "AppIconSmall")
                data = UIImageJPEGRepresentation(image!, strength)
                break
            }
            strength -= 0.1
            data = UIImageJPEGRepresentation(image, strength)
        }
        return data
    }
```    

#### 分享回调

```   
    // MARK: - SDK回调
    extension AppDelegate: WXApiDelegate {

    @available(iOS, obsoleted: 9.0)
    func application(_ application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject) -> Bool {
        if let sourceApp = sourceApplication {
            if sourceApp.contains("tencent.xin") {
                return WXApi.handleOpen(url as URL?, delegate: self)
            } else if sourceApp.contains("xxx") {
                return WKAuth.handleOpen(url as URL?, delegate: WifiService.shared)
        }
        }
        return true
    }

    @available(iOS 9.0, *)
    func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey: Any] = [:]) -> Bool {
        let appName: String = options[UIApplicationOpenURLOptionsKey.sourceApplication] as! String
        if appName.contains("tencent.xin") {
            return WXApi.handleOpen(url, delegate: self)
        } else if appName.contains("xxx") {
            return WKAuth.handleOpen(url, delegate: WifiService.shared)
        }
        return true
    }

    //未使用
    func onReq(_ req: BaseReq!) {
        if req .isKind(of: GetMessageFromWXReq.self) {
            //微信请求App提供内容， 需要app提供内容后使用sendRsp返回
        } else if req .isKind(of: ShowMessageFromWXReq.self) {
            //显示微信传过来的内容
        } else if req .isKind(of: LaunchFromWXReq.self) {
            //从微信启动App
    }
}

    func onResp(_ resp: BaseResp!) {
        let popview = PopupView()
        if resp.errCode == 0 {
            popview.popupType(.success, .shareSuccess)
        } else {
            popview.popupType(.failed, .shareFailed)
    }
}
```
