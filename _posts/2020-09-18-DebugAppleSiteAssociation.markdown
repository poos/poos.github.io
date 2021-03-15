---
layout:     post
title:      "Apple App Site Association"
subtitle:   博客发布模拟, 多应用共用
date:       2020-09-18
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Debug
---

### 介绍

记录一个 AppleAppSiteAssociation 的发布历程吧，总结一下中间的经验教训。

#### 配置流程

https://developer.apple.com/documentation/safariservices/supporting_associated_domains

基本按照流程走即可，坑位在下边...


```
{
  "applinks": {
      "details": [
           {
             "appIDs": [ "ABCDE12345.com.example.app", "ABCDE12345.com.example.app2" ],
             "components": [
               {
                  "#": "no_universal_links",
                  "exclude": true,
                  "comment": "Matches any URL whose fragment equals no_universal_links and instructs the system not to open it as a universal link"
               },
               {
                  "/": "/buy/*",
                  "comment": "Matches any URL whose path starts with /buy/"
               },
               {
                  "/": "/help/website/*",
                  "exclude": true,
                  "comment": "Matches any URL whose path starts with /help/website/ and instructs the system not to open it as a universal link"
               },
               {
                  "/": "/help/*",
                  "?": { "articleNumber": "????" },
                  "comment": "Matches any URL whose path starts with /help/ and which has a query item with name 'articleNumber' and a value of exactly 4 characters"
               }
             ]
           }
       ]
   },
   "webcredentials": {
      "apps": [ "ABCDE12345.com.example.app" ]
   },
    "appclips": {
        "apps": ["ABCED12345.com.example.MyApp.Clip"]
    }
}
```

#### App-Prefix

**ABCED12345**.com.example.MyApp.Clip

可能 Developer 账号经历了迁移，新的账号 ID 同时支持 mac 和 iOS 等应用。

这个时候就需要到网站上看具体的 bundle-id 的 App-Prefix **（不一定是账号 ID）**。

#### Debug

https://ios13.dev/universal-links-debugging-on-ios-13-cjwsux93w001p6ws1swtstmzc

1. json 部署

怎么知道文件部署好了没有，基本上是看 json 能否被设备成功解析。除了**https**的要求外，还要求**当前设备所处的网络环境无重定向，账户验证等**。

例如下边的命令是用来验证服务端部署的，就依赖电脑的网络环境。

```
curl -I https://www.uber.com/.well-known/apple-app-site-association
```

2. 手机成功解析 json 和跳转log

- 在 **app 安装**的时候（通过各种途径安装，非启动）会下载和配置 json 数据。
- 在点击 universal-link 的时候会有其他log。

这时候为了验证设置的 json 是否已经被手机认可，就可以看下设备的 log，有两种方式。

- 打开 Console.使用 `apple-app-site-association` 或者使用 `swcd` 做搜索。
- 查看手机日志 `sysdiagnose_YYYY.MM.DD_HH-MM-SS-XX…` 中的 `swcutil_show.txt` 文件。


> 还可以通过 sysdiagnose 的日志来查看。 https://download.developer.apple.com/iOS/iOS_Logs/sysdiagnose_Logging_Instructions.pdf。
log 在 iOS 设备的 `Settings > Privacy > Analytics > Analytics Data >` 名字是  `sysdiagnose_YYYY.MM.DD_HH-MM-SS-XX…`。还提醒一下的是，这是一份完整的设备日志（大概500M+），需要在电脑上查看详情。


##### 关于日志

`Settings.app > Privacy > Analytics > Analytics Data`

通常的 App crash 日志也是放在这里，通过导出日志，然后使用 *.dsym 文件即可反符号到相应的代码提升。


#### 多 domain 使用了同一个 json

沟通中发现，我们当前服务端有多个 domain 最终使用了同一个 json file。

所有的 app bundle id 都被正确的配置了。服务端虽然在不同的 domain，但是客户端根据 domain 做后续的事情，所以每个app都能正确的工作，没有问题。

**也就是说 json 中的非本应用 bundleid 的部分被忽略了。**

#### 借助博客网站验证 json

通常后端的发布都是有周期的，那么为了提前验证 json 是否正确，就可以找个网站进行验证。例如 firebase 的 `pagelink` 等。但是有更好的方式，使用 `github.io` 博客网址的 domain 也可以方便的验证 json 是否正确。

> 配置 json 到能访问的位置即可，或者 404 的page，哈哈哈。

#### 跳转

当一切都搞好之后，就可以测试 link 的 path 了，使用联系人的个人网站，Notes 等都可以测。

#### 短链接

测试中邮件有发送短链接，经过重定向然后到达指定 url：

这种情况在 iOS 没法正常的工作。

> android 可以。😂

### 最后

文章主要是将了一些坑点，具体的文档大概介绍了一下。对初次配置的小伙伴应该帮助比较大，看一下这些奇技淫巧可以省很多时间。
