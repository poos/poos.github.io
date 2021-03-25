---
layout:     post
title:      项目优化-瘦身
subtitle:   图片, 库, 代码, 项目设置, 多方面优化项目
date:       2018-06-16
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Improve
- Summary
---


---
```
18年9月更新，因为苹果AppStore的bug导致显示的大小于Itunes Connect显示大小不一致，9月底已经修复。
```
---


本文主要介绍项目瘦身，代码精简方面的优化，列了一些步骤可以对照修改。

---

总结了以下优化大概可以归为以下几类：

1. 图片等资源文件优化（无用/重复/压缩/矢量图_有待证实/存服务器等）

2. 第三方库的优化，(oc下可避免use_frameworks！）

3. 代码方面的优化

4. xcode设置和支持bitcode


---

以下正文，不推荐的方式标题等级会小一些

---


## 1. 图片处理

### 1.1 查找没用到的图片支持Swift和OC
https://github.com/tinymind/LSUnusedResources/

### 1.2 查找MD5一样的文件（图片去重复）
https://linux.cn/article-6127-1.html

https://github.com/adrianlopezroche/fdupes
```
isntall from  http://macappstore.org/fdupes/
    cd xx
    fdupes -Sr ./
```

#### 1.3 大图转换格式

对于超过10KB的大图，不管是webp格式还是jpg格式都比png小很多，png转换webp的工具可以采用：
```
# 安装
brew install webp
# 转换 webp
cwebp -q 75 bts-home-nocity@3x.png -o bts-home-nocity@3x.webp
```

### 1.4 图片简单压缩。
正常压缩图片即可。满足使用。

无损压缩，去除图片data的无用部分，待证实对iOS是否有效？（打包过程中 Xcode会把之前处理过的逆转回来？）

https://github.com/ImageOptim/ImageOptim

### 1.5 assets文件正确使用
https://www.jianshu.com/p/867d9fa85770

### 1.6 非必须图片从服务器下载
如标题：非必须图片从服务器下载。

## 2. 第三方库

### 2.1 三方库的选择
检查三方库的大小，权衡是否使用，例如使用Realm性能提升，但是打包大小会多7M左右

OC下还可以鉴别使用use_frameworks!

#### 2.2 三方库直接拉入项目，使用三方库的部分内容
不使用cocoapod等三方管理，少了三方的配置文件，但是牺牲了管理代码方遍性

拆分三方库的代码，仅使用部分，谨慎

## 3代码去重复和删除
### 3.1推荐AppCode分析重复/未使用的代码
https://www.jetbrains.com/objc/download/
下载地址

#### 3.2静态代码分析工具PMD下的CPD，分析重复代码

https://github.com/pmd/pmd

https://www.jianshu.com/p/721ca80fdcc7

```
//安装PMD

$ cd $HOME
$ curl -OL https://github.com/pmd/pmd/releases/download/pmd_releases%2F6.4.0/pmd-bin-6.4.0.zip
$ unzip pmd-bin-6.4.0.zip

//这里要求安装jre，参看下边
$ alias pmd="$HOME/pmd-bin-6.4.0/bin/run.sh pmd"
$ pmd -d /usr/src -R java-basic -f text

```


```
//安装Jre，修改环境变量
//No Java runtime present, requesting install.
//安装完还提示这个错误，可以

vim .bash_profile
添加：  
export JAVA_HOME="/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home"

export PATH=${JAVA_HOME}/bin:$PATH
保存，并关闭
然后  source .bash_profile

java -version

```
开始检查

pmd文档链接如下

https://pmd.github.io/pmd-6.4.0/pmd_userdocs_cpd.html#options
```

 cd pmd-bin-6.4.0/bin/

 ./run.sh cpd --minimum-tokens 100 --files /Users/bieshixuan/iOS/ZhiMa_iOS --language swift --format xml
// 可以导command K清除之后保存
```


-----ka
#### pmd cpd --files...命令报错，可以手动替换上边的内容
安装好 PMD 后, 就可以在 Xcode 中添加 Run Script 脚本了:
```
# Running CPD
//如果错误，可以注释这一行，手动替换
pmd cpd --files ${EXECUTABLE_NAME} --minimum-tokens 50 --language swift --encoding UTF-8 --format net.sourceforge.pmd.cpd.XMLRenderer > cpd-output.xml --failOnViolation true
# Running script
php ./cpd_script.php -cpd-xml cpd-output.xml
```
需要的 cpd_script.php 文件如下, 需要放到工程根目录
```
<?php
foreach (simplexml_load_file('cpd-output.xml')->duplication as $duplication) {
    $files = $duplication->xpath('file');
    foreach ($files as $file) {
        echo $file['path'].':'.$file['line'].':1: warning: '.$duplication['lines'].' copy-pasted lines from: '
            .implode(', ', array_map(function ($otherFile) { return $otherFile['path'].':'.$otherFile['line']; },
            array_filter($files, function ($f) use (&$file) { return $f != $file; }))).PHP_EOL;
    }
}
?>
```

还有更多静态代码分析工具查看http://hao.jobbole.com/static_code_analysis_tool_list_opensource/


#### OC下可以检查没有使用的import
```
OC下发现没有使用的import

https://github.com/dblock/fui
```
```
有一个现成的工具，就是fui，非常好用，以前可以直接添加插件使用，但是现在大家都知道，xcode不能使用第三方的插件了：gem install fui。但是就这一句都经常会出现问题，比如

sudo gem install -n /usr/local/bin fui
运行完成后会直接在终端展示出很多没用的代码，有的是import了但是没有使用的，也有目录下有但是工程中没有添加到也没有用到的，最好还是全局搜索一下再删除。

```

#### 3.3 文件之间相似度
如果很相似可以权衡是否合并继承

https://github.com/startry/SameCodeFinder

```
//越小相似度越高
python SameCodeFinder.py ProjectPath .swift --detail
```

#### 3.4 检查代码其他方面

1. 已经下线的陈旧代码，AB试验已经下线的代码

2. 通过转H5、Hybrid或者RN实现的Native功能，可以定期清理

3. 一些非核心Hybrid或者RN模块，可以考虑不要打包进入APP，通过动态下发的方式获取

4. 代码的重构，UI组件、业务逻辑的重用等等

5. 结合项目的混淆，缩短类名方法名长度来减少大小

6. 使用包含所有内容的Model，将多个file代码整合到一个file中，同时也会加快编译速度



## 4. Xcode设置和bitcode

### 4.1 Xcode（使用新版xcode可以跳过）
```
Build Settings->Optimization Level
有几个编译优化选项，release版应该选择Fastest, Smalllest，这个选项会开启那些不增加代码大小的全部优化，并让可执行文件尽可能小。
2.去除符号信息
Strip Linked Product / Deployment Postprocessing / Symbols Hidden by Default
在release版本应该设为yes，可以去除不必要的调试符号。
Symbols Hidden by Default会把所有符号都定义成”private extern”，详细信息见官方文档。

这些选项目前都是Xcode里release的默认选项，但旧版XCode生成的项目可能不是，可以检查一下。其他优化还可以参考官方文档—CodeFootprint.pdf
```

### 4.2 bitcode 可以减小打包的大小
默认Xcode是true的，如果因为第三方库的原因改为了false，可以向库支持者寻求帮助，而且开启之后可以线上现在dSYM文件不用本地保存了

---

以下是优化点的参考资料

---
Damonwong的对yep优化分析

https://www.jianshu.com/p/df52554be812

iOS可执行文件瘦身方法

http://blog.cnbang.net/tech/2544/


总结下在iOS瘦身上的一些方向和优化点

https://www.ctolib.com/topics-65758.html


---

END ，以下是一些其他地方可能用到的

---
---
#### 通过遍历本地的图片名，查找在文件中是否使用了图片名
##### shell命令, 不能查找assets中的json名字
```
通过终端 执行 shell 命令

a. 第一步建立.sh 文件  如 unusedImage.sh

#!/bin/sh  
PROJ=`find . -name '*.swift' -name '*.xib' -o -name '*.[mh]'`  

for png in `find . -name '*.png'`  
do  
    name=`basename $png`
    if ! grep -qhs "$name" "$PROJ"; then  
        echo "$png is not referenced"  
    fi  
done


b. 进入你要查找的工程目录下执行 这段 shell 脚本

sh unusedImage.sh   

```


##### 搜索文件，搜索某个字符串在子文件夹是否使用了
搜索文件的方式可以使用grep,ack都是不错的工具，但是有一种更快，更好的搜索文件内容的方式:The Silver Searcher，The Silver Searcher使用起来更方便，更快，更简单,项目地址：https://github.com/ggreer/the_silver_searcher。 直接安装The Silver Searcher的命令:
```
brew install the_silver_searcher
```
使用ag命令就可以进行文本搜索:
```
ag "image" './'
```
这个命令的意思是搜索到该目录下以及其子目录下的所有含有"image"的文件。
使用这个命令就需要在python中执行bash命令。



### 参考资料

github项目：

```
ImageOptim
LSUnusedResources
SameCodeFinder
SMCheckProject
XcodeProjectArrangementTool
```
博客：


[iOS Color Misaligned Images优化 - 简书](https://www.jianshu.com/p/38cf9c170141)

[浅谈iOS中的视图优化 - 简书](https://www.jianshu.com/p/5c968a240e27)

[图片的颜色深度/颜色格式（32bit,24bit,12bit） - 简书](https://www.jianshu.com/p/52440c7a8902)

[UILabel在iOS8下的Color Blended Layers - 简书](https://www.jianshu.com/p/db6602413fa3)

[如何改善图形性能 | 蘑菇味海星](http://ihomway.cc/2017/07/28/ios-grapics-performance/)

[怎么改png图片位深度 小小知识站](http://www.zhishizhan.net/wuhuabamen/43560.html)
