---
layout:     post                       # 使用的布局（不需要改）
title:      客户端的外部调试：JS 调试，应用互调调试，抓包和代理调试                 # 标题
subtitle:   App web 调试，Other web 调试，Universal Links 和 App scheme 配置和测试，Charles https 抓包 和 代码中设置 https Proxy           #副标题
date:       2018-11-12                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 工具
---

**[本文所有 工具 和 代码 地址](https://github.com/poos/BlogDemo)**

### App 中的 web 和 Safari 中的 web 调试

在开发中经常会碰到跟 H5 页面联调的情况，这种情况下所有的操作并不能在我们的 IDE 中检测到，那么如何调试呢，本章会详细介绍。Get 到了本文的技能之后，React 调试会更加方便，会更加一针见血的调试到问题所在。

![img](https://upload-images.jianshu.io/upload_images/2363263-9b5f0e4ce311e25b.jpg)

这是一个入门的教程：[Safari 真机调试js](https://www.jianshu.com/p/ed4b1bfb57dc)

基本步骤如下：

1. 开启iOS设备的调试功能，打开“设置”程序，进入“Safari”->“高级”页面开启“Web检查器”

2. 开启电脑 Develop 功能，点击“Safari”菜单下面的“偏好设置（Preferences...）”，切换到“高级选项（Advanced）”，选择 develop

3. 连接设备，点击电脑 Safari 菜单 Develop 下的设备名，调试的 web 名（ 注意 APP 内部的 web 调试需要提前打开页面）

**教程中没有提到的：**

1. 网页检查器和 safari 中的 "command + option + I" 是一个调试器，所以功能还是很强大的

2. 选择 “网络” 菜单，可以查看请求头，请求体，请求结果等，可以查看 App 是否正常提交 token ，user agent 是否正确提交等

3. 元素页面可以编辑调试 html 页面，h5 开发用的到

4. 调试器最下边一行是 **命令行** ，可以执行操作，用于测试调用原生，页面跳转，互调等非常方便。


### Universal Links 和 App scheme 配置和测试

如果需要从 浏览器 跳转 App，有必要了解一下本章，简短说，通常情况下只需要配置 **Universal Links** 和 **Scheme**

#### Universal Links 跳转 APP

**[iOS9 Universal Links踩坑之旅，移动应用之deeplink唤醒app](https://www.jianshu.com/p/77b530f0c67b)**

上边的链接是一个很好的教程。值得一提是文章介绍了如何 **在自己 APP 中限制其他 APP 的 link** （仿微信的限制）。这里就简单总结一下，如何配置 Universal Links：

1. 创建 apple-app-site-association 文件并且放在服务器上

2. 配置app 的 applink，然后在app里面添加代理方法

#### App scheme 跳转 APP

上边的方式可以在其他应用 **使用 Safari 打开** 或者 **从其他域名网站跳转到本 APP 域名网页** 时候检查并且自动跳转至 APP。

另外一种情况是，我们已经安装 APP，在 Safari 中浏览（safari 顶部会出现在 APP 中打开），但是我们 H5 想要主动调起 App ，这就需要 Scheme 了。

1. app info.plist 设置 Scheme

2. 代理方法监听

```
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey: Any] = [:]) -> Bool {
        let appName: String = options[UIApplicationOpenURLOptionsKey.sourceApplication] as! String
        if appName.contains("tencent.xin") {
            return WXApi.handleOpen(url, delegate: self)
        }else if url.absoluteString.contains("zhimawenda://jump") {
            jump...
        } else {
            UIApplication.shared.openURL(url)
        }
        return true
    }
```

#### 如何调试

非常简单，参考上一章的 方法

1. 打开 iphone safari， 打开电脑相应的调试器

2. 在最下边命令行输入命令调试即可

```
//使用 iframe 和 widow 跳转都可以
location.href = "scheme://xxxx"
```

### Charles https 抓包

![img](https://upload-images.jianshu.io/upload_images/2469183-c422573ff7075723.png?imageMogr2/auto-orient/)

上边第二章 Universal Links 实现时候我们需要检查 app 首次启动时候，apple-app-site-association 文件是否下载成功。这个时候就需要 Https 抓包了。

这也是一篇比较好的文章：[抓包之Charles For Mac 4.0+破解版](https://www.jianshu.com/p/1c1023036a75)

同样简单总结：

1. 安装电脑版和破解（怕折腾和土豪跳过）

2. 手机连接同一局域网，设置代理地址和端口 8888 ，访问下载 ssh 证书

3. **重要，到手机 通用 -> 关于本机 -> 证书信任设置 信任证书** [IOS手机为什么安装Charles证书后，还是抓不到HTTPS请求](https://www.jianshu.com/p/3b26934fb839)

完成之后 就可以调试了；同样可以调试其他 API 的请求。推荐去 Charles 的 Recording Setting -> Include 设置过滤：

```
https://*.xxx.com
```

![img](https://img.yzcdn.cn/public_files/2018/04/18/1450c9728e75cbbe6f6532370cc36ecf.png)

### 后续：代码设置代理


**解决缺点** ，每次调试都需要打开 iPhone 的设置，打开代理，写地址和端口；调试完成之后又需要关闭。有没有代码的方式设置呢：

我在 [stackoverflow](https://stackoverflow.com/questions/28101582/how-to-programmatically-add-a-proxy-to-an-nsurlsession/37466532) 找到了如下的代码：

```
let proxyHost = "192.168.1.226"
let proxyPort = 8888
let configuration = URLSessionConfiguration.default
configuration.connectionProxyDictionary = [
    kCFNetworkProxiesHTTPEnable: true,
    kCFNetworkProxiesHTTPProxy: proxyHost,
    kCFNetworkProxiesHTTPPort: proxyPort,
]
/* 下边的键值对已经不可用
kCFNetworkProxiesHTTPSEnable: true,
kCFNetworkProxiesHTTPSProxy: proxyHost,
kCFNetworkProxiesHTTPSPort: proxyPort
*/
```

**将 URLSessionConfiguration配置在你封装的网络框架中即可**

但是很快发现 **关于 https 的键在 swift 下已经不可用， 在 OC 下运行正常** 。

经过多方寻找，我找到了国际友人写的这个：[Debugging iOS Simulator network calls (for free)](https://up.smartrecruiters.com/debugging-ios-simulator-network-calls-for-free-b8e02a0c9bed)，简直就是蓦然回首，灯火阑珊，没想到远在大陆彼岸的友人会有跟我相同的想法。还比我更早一步，nb.

将上边的代码替换到下边即可, 原理是直接找了 key 对应的字符串的值，而进行设置

```
let proxyHost = "192.168.1.226"
let proxyPort = 8888
let configuration = URLSessionConfiguration.default
configuration.connectionProxyDictionary = [
    "HTTPSEnable": 1,
    "HTTPSProxy": proxyHost，
    "HTTPSPort": proxyPort
]
```
过程是曲折坎坷的，收获也是不错的，终于 **能单独在我们的 app 中设置代理** ，也不用去设置中心来回修改了。

**不过记得要把这段代码添加 DEBUG 宏判断里面去** 或者 **设置专门的 config**，以防止在正式环境出现错误


## 总结

是不是看了本文之后就像又打开了一扇门，借助这些工具我们调试就能够切中痛点，一击致命，所向披靡，无所畏惧...

最后本文的 Charles 工具同样会放于文章开头的地址，帮到你的话点个赞吧。
