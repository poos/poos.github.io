---
layout:     post                       # 使用的布局（不需要改）
title:      "Terminal"                   # 标题
subtitle:   FileMode, Mov->Mp4, Image compression and convert...             #副标题
date:       2020-06-07                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- Terminal
---



### 背景

就是简单列举一些用到的脚本命令，可以 save your life。


#### brew

先安装这个，然后就可以安装后续各个命令行了

```
brew -v

brew search fastlane
```


#### mas

登陆 app store 之后可以根据`appid`用命令行直接下载app

```
mas install xx
```

#### p4v - git: fileMode

在使用 P4v 的时候，文件夹基本是加锁的，通常会解锁和修改代码/运行脚本等，像下边一样：

// change mode 111 111 111
```
cd xx
chmod -R 777 .
```

而 P4v 不被 Xcode 自动识别，所以为了能够快速的看到代码修改，就创建一个本地的 git，这时候问题就来了。

当文件权限修改的时候，git 会认为所以的文件修改过了，这时候就可以使用下边的命名进行忽略：

```
git config core.filemode false

git config --global core.fileMode false  
```

#### Mov -> Mp4

通常公司一些 Wiki 网站不支持 mov 格式（并不是所有人都是mac电脑），这时候传视频的时候可以考虑将 mov 转换 到 mp4 上传～

首先安装`ffmpeg`， 使用`brew`即可安装～

运行下面命令即可：
```
ffmpeg -i input.mov -acodec copy -vcodec copy output.mp4
```

#### Convert

将多个屏幕截图合并到同一个截图。这在对比一些截图效果的时候比较好。

首先安装`imagemagick`， 使用`brew`即可安装～

运行下面命令即可：
```
convert a1.png a2.png a3.png +append r.png
```

#### Compression

还是图片，通常图片占用空间太大，做博客或者Wiki的时候缩小图片大小的时候，这个就有用了。

首先安装`pngquant`， 使用`brew`即可安装～

运行下面命令即可：
```
cd xx
find . -name '*.png' | xargs pngquant --quality=10-50
```

##### 附赠一个 Compression & Convert


```
echo "re-mkdir new/"
rm -rf new/
mkdir new/

echo "pngquant imgs"
find . -name '*.png' | xargs pngquant

echo "imgs move to new/"

find . -name "*-fs8.png" | while read f
do
  mv "${f}" "new/${f/-fs8/}"
done

echo "convert new/"
convert `find new/ -name '*.png' | sort` +append new/r.png

```

### 最后

这篇主要是 brew 里面和一些常用的脚本的分享，如果有好的还要继续添加...

PS:现在使用 swift 也可以编写脚本了，创建 Command Line project，一步步code即可。而且能打包放到`usr/bin`使用，等新的文章在做分享吧。
