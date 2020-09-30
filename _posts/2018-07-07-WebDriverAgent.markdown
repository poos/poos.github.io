---
layout:     post                       # 使用的布局（不需要改）
title:      使用WebDriverAgent学习其他app布局,自动化测试               # 标题
subtitle:   通过脚本化启动 WebDriverAgent， 使用ATX自动化测试 ； 自动微信跳一跳等               #副标题
date:       2018-07-07                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 工具
---

---
**19.7.5更新**

新版本可能需要 重新运行脚本更新项目环境依赖。’./Scripts/bootstrap.sh‘



**end**

---
---
**18.12.26更新**

最新版已经支持 Xcode 10 和 iOS 12 ，重新拉取最新代码即可。而且最新代码优化了一些其他的UI体验，不需要返回界面黑屏重新查看了。

真机测试更改 Build Setting 中的 product bundle identifier 为 自己的，自动创建证书即可测试。

**end**

---
---
**9月25更新**

### 使用最新的Xcode10正式版没法正常测试，可能是某些库的问题，但是可以在9.4下添加iOS12支持包即可

[添加iOS支持包](https://www.jianshu.com/p/1a33e36c4b67)

ps: Xcode升级时候先邮件复制一份旧的，然后直接appStore更新，不用慌。

**end**

---

### 有太多的文章介绍WebDriverAgent安装配置，本文着重介绍使用command line命令行快速调起


#### [WebDriverAgent下载地址](https://github.com/facebook/WebDriverAgent)

[配置博客地址](https://blog.csdn.net/PRIMEFJT/article/details/78947480)

#### 安装简介

```
如果你电脑上没有安装Homebrew，使用下面的命令安装：

/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


如果没有安装carthage，使用下面的命令安装：
brew install carthage

然后按照Python环境。
brew install python


安装Node环境，命令如下：
brew install node


下载WebDriverAgent
git clone https://github.com/facebook/WebDriverAgent


下载完毕后，进入到 WebDriverAgent 目录，执行如下脚本。安装相关工具
cd ./WebDriverAgent/
//执行脚本
./Scripts/bootstrap.sh

```

```
有些国产的iPhone机器通过手机的IP和端口还不能访问，此时需要将手机的端口转发到Mac上。

$ brew install libimobiledevice

iproxy 8100 8100
```


#### 开始使用WebDriverAgent
```
# 解锁keychain，以便可以正常的签名应用，
PASSWORD="******"
security unlock-keychain -p $PASSWORD ~/Library/Keychains/login.keychain

# 获取设备的UDID
UDID=$(idevice_id -l | head -n1)

# 运行测试
xcodebuild -project ~/WebDriverAgent-master/WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination "id=e6a545a6a7490d06e3f5eb32e6f3a6d843ac2d08" test

xcodebuild -project ~/WebDriverAgent-master/WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination "id=278c6390c9e6b372fd4b4bdd737022e2c6ff9dcd" test

# 手机IP的端口映射到本地
iproxy 8100 8100

# 端口占用取消
ps -ax|grep -i "iproxy"|grep -v grep|awk '{print "kill -9 " $1}'|sh

# 打开safari
open -a Safari http://localhost:8100/inspector
```





---

##### 通过ATX(AutomatorX)进行界面测试

```
ATX安装和使用

ATX(AutomatorX的简称）的安装比较简单，主要有两个命令。

pip install --pre --upgrade atx
pip install opencv_python




ATX的编写都在 python 实现，例如：

import atx
d = atx.connect('http://localhost:8100', platform='ios')
print d.status()

＃命令行执行
python test.py

```



---

##### 通过wechat_jump_game自动微信跳一跳

就不放链接了，我基本上用**开始使用WebDriverAgent**这一模块快速开始界面查看
