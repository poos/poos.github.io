---
layout:     post                       # 使用的布局（不需要改）
title:      局域网发布ipa （类似蒲公英，fir 等）                # 标题
subtitle:   借用 github存储 manifest.plist; Build/xx.app 脚本导出 ipa; 借用博客在局域网发布...             #副标题
date:       2018-10-30                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 工具
---


**目前可以在局域网和 github 访问下载 ipa，可以从日常 build 导出ipa（未加密，应只在公司网使用）**

## to do

一键打包 build 中的 app，上传ipa本地发布，甚至发消息提示测试
一键 achieve app，上传ipa本地发布，甚至发消息提示测试

[通过xcodebuild自动构建并发布Ad Hoc测试包](https://daniate.com/2016/11/27/通过xcodebuild自动构建并发布Ad-Hoc测试包.html)

## 踩坑点

10月30日

1. **plist 要在 https:// 上面**

2. **plist 中的链接要准确获取：ipa为下载链接，image同样，bundle-identifier要准确，bundle-version和title可以不准确**

3. **ipa 下载地址选择：优先选择局域网；github上传大文件使用Git Lfs（不推荐下载慢）；七牛下载（未正确获取下载链接，未测试）**

4. **导出 ipa 方式选择：xcrun + PackageApplication（在日常 build 中导出ipa）；脚本打包 achieve，导出出 ipa**

```
xcrun -sdk iphoneos PackageApplication -v ~/xx/xx.app -o ~/xx/xx.ipa
```

5. **在局域网网页发布：使用 xxx.loacl:4000/ 网址；个人借用了博客 jekyll 新建了个发布静态网页**

```
jekyll serve -w --host=0.0.0.0

http://xx.local:4000/test/app
```


## 原理和分析

iOS 的 ipa 可以通过链接（ex：itms-services://?action=download-manifest&url=https://xxx/xx.plist）去请求 itunes 安装app（没有静态网页直接请求这个链接，或者生成二维码在相机扫码即可下载 ipa）。

plist 文件格式如下： 有两个对象一个存储 iap 和图片，另一个对象存储 app identifier 和 版本信息，plist 代码稍后在下边列出。
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>http://xxx/test.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>url</key>
					<string>http://xxx/icon57.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>url</key>
					<string>http://xxx/icon512.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>com.appid</string>
				<key>bundle-version</key>
				<string>1.0</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>XXName</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

## 过程

### github 发布ipa

**如果不能正常打开，检查下边几项**

1. plist 文件： ipa 是否是下载地址；bundle identifier 是否正确

2. plist 文件是否能正常访问

3. ipa 是否对应 bundle identifier

4. 如果在 静态 blog 中，不要使用 xx.io/xx.ipa；itunes 有可能拿不到

5. 还是不行，新建 repo 去存储 manifest.plsit

参考资料

[利用Github 实现iOS 测试包的在线分发](https://www.jianshu.com/p/b885f14b00e2): Github 存储 plist + Flask 搭建静态网页

[iOS 用github自建ipa应用分发平台](https://www.jianshu.com/p/3f9e11fad442)

[1ilI/TestMyipa_Resource](https://github.com/1ilI/TestMyipa_Resource)


### github 上传ipa超过100M

注意是要获取下载ipa的地址

**不建议使用 github 存储 ipa** ，访问速度很慢

[关于Github的上传文件超限问题解决](https://blog.csdn.net/qq_35559420/article/details/81783787)

**局域网搭建好了，去除 git LFs**

删除 .gitattributes, 添加忽略*.ipa，删除 app.ipa, push

如果 push 不成功，可能是本地有多个 commint ，其中有 app.ipa 的修改，不需要后退合并 commint 即可，需要先 push 那些分支在删除

**ps：全局git 忽略文件**

[使用Git如何优雅的忽略掉一些不必的文件](https://www.cnblogs.com/zuopeng/p/4305367.html)

第一步要找到一个 .gitignore_global 的配置文件,在~/ 目录下使用ls -all就能找这个文件,然后vi .gitignore_global打开它,把下面的代码添加进去,然后wq退项目.

```
# Xcode
#
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata
*.xccheckout
*.moved-aside
DerivedData
*.hmap
*.ipa
*.xcuserstate

# CocoaPods
#
# We recommend against adding the Pods directory to your .gitignore. However
# you should judge for yourself, the pros and cons are mentioned at:
# http://guides.cocoapods.org/using/using-cocoapods.html#should-i-ignore-the-pods-directory-in-source-control
#
# Pods/
```

###  打包 ipa 的 N 种方式

博客资料很多，其实究其原理两种

1. achieve 然后导出 ipa

2. 将 build 的.app 转换为 ipa

抛却 zip，手动打包等方式。如果完整打包就使用 **xcodebuild -exportArchive**，这也是 Xcode8 之后御用的方式；

#### 如果只是需要将 .app 导出为 .ipa 可以使用 Xrun

**使用Xrun将 build 直接转换为 ipa，因为打包块，直接把平时运行的 build 导出 ipa 了**

Xcode 8 以上即不包含 PackageApplication 包，添加方法，注意 **sudo**
```
# 下载
https://download.csdn.net/download/nb_token/10363097

添加PackageApplication

# 放到指定目录
sudo cp ./PackageApplication /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/

# 终端执行下面两条命令
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/

sudo chmod +x /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/PackageApplication
```

[iOS自动构建以及打包命令(xcodebuild,xcrun)](https://blog.csdn.net/skylin19840101/article/details/54406080)

[Jenkins集成iOS自动化打包（GitLab + xcodebuild + xcrun + ftp）](https://blog.csdn.net/NB_Token/article/details/80018747?utm_source=blogxgwz7)

[Xrun 将 app 转化为 IPA](https://www.cnblogs.com/kingbo/p/3709617.html)

[Xcode脚本自动化打包问题xcrun: error: unable to find utility "PackageApplication", not a developer tool or in PATH](https://www.cnblogs.com/Crazy-ZY/p/7115076.html)

**打包命令**

```
xcrun -sdk iphoneos PackageApplication -v ~/xx/xx.app -o ~/xx/xx.ipa
```

### mainfest 生成

在原理分析一章已经介绍过了

重申一下，**.plist 中所有的链接都要是直接指向文件的链接**

### ipa 下载

只需要在手机 safari 中 访问如下链接即可

```
itms-services://?action=download-manifest&url=https://xxx/xx.plist
```

**直接生成二维码，相机扫码访问**

**在静态网页中添加 <a> 链接访问**

#### jekyll 静态博客发布到局域网

上代码

```
// 发布
jekyll serve -w --host=0.0.0.0

// 端口占用
ps -ax|grep -i "jekyll"|grep -v grep|awk '{print "kill -9 " $1}'|sh

```

另外

在 mac 中设置电脑的 name.local 即可使用 name.local:端口//xxx 访问，这样就不用去查IP了

```
偏好设置 -> 共享 -> ”电脑名下边的 local 地址”
ex:

http:shown.local:4000/test

```
[如何让jekyll服务可以在局域网中访问](https://www.jianshu.com/p/650b96306013)

#### jekyll 静态博客 的其他使用

**屏蔽某些页面**

我在 nav 的 遍历博客中添加了博客 title 的判断，如果是 'test' 就不会出现在博客目录中

```
{% if page.title != 'test') %}
```
现在我有两个页面，一个测试 scheme 码的测试页面，一个是 ipa 的下载页面

**静态 json 文件**

注意一下访问地址

```
https://raw.githubusercontent.com/poos/poos.github.io/master/test/xx.json
```


### 结语

正常配置即可访问安装了。测试设备的100个设备现在装起来就很方便了。

过程算是走通了，现在算是半自动化吧，后续再看看脚本完善优化一下。
