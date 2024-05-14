---
layout:     post
title:      Gitlab 一行配置
subtitle:   通过一行简单的配置即可接入CI
date:       2022-04-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- CICD
- Toolchain
- Linter
---

## 背景

使用 gitlib 可以自动化的进行仓库检查、仓库发布、自动提交主项目、制作二进制（优化build时长）等任务。通过脚本也可以做到定时打包，配置项同步数据库等。

但是不合适的使用，也未必能给大家减轻负担。比如：仓库太多，每个仓库的 `.gitlab.yml` 都要手动配置？本文以此为开始

### `include`
话不多说，了解一下`include`。

**`.gitlab.yml`本身支持一种`include`的方式，继承其他`yml`。**

也就是说，以前每个job都要自己写一套，现在好了，直接一行代码引用（呼应标题）。

**除了方便，还有其他好处吗？**

- 稳定，通常的 CI 方案，每次执行任务都需要去拉取脚本仓库，那么 gitlab 不稳定时候会有大量 timeout；
- 可修改，有的一些脚本，执行效率低，理论上结合当前我们项目的使用，可以优化执行任务；各个仓库自己使用各自`.gitlab.yml`配置，没法同步新功能；
- 可配置，全局选项等；
- 日志，可监控，随时调整；

其他仓库通过继承其他`yml`，无需关心其他，内部优化可走起来，大干一场了。


## 知识储备

### yml

- 首先储备一下 yml 的基本文档，知道一下当前安装版本的 gitlab 能够做哪些事情：

[gitlab-readme](https://code.team.com/help/ci/yaml/README)

1. 除了常见的`script`, `stags`, `tags`，还要知道 `before_script`, `after_script`, `only`, `except`, `artifacts`, `variables`等参数。
2. 还有一些 `include`, `extends` 这样的层级嵌套问题。

- 常见的参数需要知道它运行的环境，支持的参数。即可，然后绕过一些坑点即可。
`only`要特别注意一下支持的参数类型，因为任务通常通过 only 来执行。

[gitlab-predefined_variable](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)

[where_variables_can_be_used](https://code.team.com/help/ci/variables/where_variables_can_be_used.md)

### 设置 runner

runner 比较独立，只在这里提一嘴，为了对得起标题 “一文学会配置 Gitlab yml”🤣，一些具体可以可以查看下面官方文档，或者 google 百度。

[gitlab.com/runner/install/osx](https://docs.gitlab.com/runner/install/osx.html)
[gitlab.com/runner/configuration/macos_setup](https://docs.gitlab.com/runner/configuration/macos_setup.html）

### 安装和升级 gitlab
这块咱也没搞过，也不敢说，拉个文档吧。
[gitlab.com/ee/install/](https://docs.gitlab.com/ee/install/)


> 不过要是想发外网，倒是有个低成本的路子。首先得有个服务器，就搞自己家电脑上吧。 1. 搞公网ip，给联通客服打电话，2. 搞域名，一些二级域名是免费dns的，挂上自己的ip，3. 几块买个域名，转到刚才的二级域名

这块显然也不是文章重点，转到`.gitlab.yml`文件来吧👇。

## YML

那加快一下进度，对于基础知识储备通过上面的文档备查即可，下面直接通过场景来看下如何使用：

### 目标
首先明确一下目标：
目标是大家不同 group 的所有仓库，能够用同一份 yml 文件，并且支持背景中的那些设想：

.gitlab.yml
```yml
include:
  - project: 'group/project'
    file: 'component_yml.yml'
```


### runner 分流

> 针对的是牛逼plus的多台机器冗余负载。

场景是，有不同的项目在不同的组内，要根据组的不同分配相应的执行机器：
```yml
stages:
  - check
  - check-1
  
check:  
  script:
    - xx
  tags:
    - RunnerTAG1
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group1/i

check-1:
  script:
    - xx
  tags:
    - RunnerTAG2
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group2/i
```

如下，还是比较简单的，通过`variables:`来分配即可。值得一提的是这里的参数只能使用 Gitlab 提供的参数，因为此时 runner 没分配，所以自定义的参数并没有执行。这个还是有一定局限。

看到上面有个弊端，实际同一个执行脚本需要写两遍。那事实上我们有更多个项目组，那就是乘数层级的上升。

有注意到上面 **知识储备** 部分有个`extend`参数，这个可以解决一部分这个问题。那就一步到位在加上`include`参数吧，直接拆分文件即可，通过 yml.yml 即可知道需要执行的任务。

yml.yml
```yml
include: "yml2.yml"
stages:
  - check

.check:
  stage: check
  script:
    - xx
```

yml2.yml
```yml 
check:  
  extends: .check
  tags:
    - RunnerTAG1
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group1/i

check-1:
  extends: .check
  tags:
    - RunnerTAG2
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group2/i
```

坑点1，有些版本`include:`不支持嵌套，所以就不能使用这种方式，或者在使用时候同时`include:`两份文件。
坑点2，目前使用的`extends:`不支持嵌套。下标有一些讨论：
multiple extends support
https://gitlab.com/gitlab-org/gitlab-foss/-/issues/53134
https://gitlab.com/gitlab-org/gitlab-foss/-/merge_requests/26801

那，**升级 gitlab**之后，第二份文件会更简洁：

yml2.yml
```yml
.group1:
  tags:
    - RunnerTAG1
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group1/i

.group2:
  tags:
    - RunnerTAG2
  only:
    variables:
      - $CI_PROJECT_PATH =~ /group2/i

check:  
  extends: 
    - .check
    - .group1

check-1:
  extends: 
    - .check
    - .group2
```

小小的改变之后，后续增加任务会简洁很多，所以用新版还是香的。


### 参数影响 job 执行

[where_variables_can_be_used](https://code.team.com/help/ci/variables/where_variables_can_be_used.md)
理论上可以使用如下的方式，但是这样就需要定义`variables: CUSTOM_VAR` 在 `.gitlab.yml` 配置相关参数。

因为篇幅较长，下面就以最简参数示例了：
```yml
variables:
  CUSTOM_VAR

check:
  only:
    variables:
      - $CUSTOM_VAR =~ /group2/i
```

那配置的参数基本是不统一的，所以就没发使用这样的方式来动态增加减少 job 项。

那另外一种退求其次的方法是：
```yml
before_script:
  - "export CUSTOM_VAR=`sh`"

job2:
  script:
    "if [[ $CUSTOM_VAR == true ]]; then\n  shxxx.sh;\nfi"
```

这部分可以追一下最新的 Gitlab，是有可能有新支持的。毕竟在 Gitlab 主服务上是可以执行一些内容的。


**另外如果出发任务不能满足你的需求和科研配合 except 使用。**例如默认的 only 是 [tags, branches]，所以可使用如下，只在 tag 上运行。

```
  except:
    - branches
```

### shell 脚本可玩性

这里简单介绍一下多行的 shell 在 gitlab的 yml 怎么写，官方文档其实有部分描述：

> NOTE: Note:
Sometimes, script commands will need to be wrapped in single or double quotes.
For example, commands that contain a colon (:) need to be wrapped in quotes so
that the YAML parser knows to interpret the whole thing as a string rather than
a "key: value" pair. Be careful when using special characters:
:, {, }, [, ], ,, &, *, #, ?, |, -, <, >, =, !, %, @, `.


简单来说在终端是这样写的
```shell
if [ -d xxx ]; then\
  cd xxx; git add .; git stash; git pull;\
else\
  git clone xxx.git xxx;\
fi
```
在 yml 需要这样写
```yml
"if [ -d xxx ]; then\n  cd xxx; git add .; git stash; git pull;\nelse\n  git clone git@xxx.gilab/a-shell.git xxx;\nfi"


sh '''if [ -d xxx ]; then\\
  cd xxx; git add .; git stash; git pull;\\
else\\
  git clone git@xxx.gilab/a-shell.git xxx;\\
fi'''
```

### 运行沙箱

关于job，有几个规则是默认的~~，即默认有几个坑点~~
1. 同一个 tag 名，有两台实际机器注册。如果连续5个任务指定了这个runner tag，那么并不保证在同一台机器执行。即，**负载均衡规则**。
2. script, before_script 等脚本默认是 job 级别的沙箱环境。下边附加例子。
3. 沙箱规则，运行完成会清理脚本相关文件和目录，不会带到下一个job。所以即使实际在同一台机器的相邻 job 也不能共享文件。

顺序执行 job 1， job 2，位于同一个runnner tag id机器上。
job 1 (tagid=0001)
```yml
before_script:
  - pwd
  # 输出结果为 xxx
  - cd A
  - pwd
  # 输出结果为 xxx/A
script:
  - pwd
  # 输出结果为 xxx/A
```

job 2 (tagid=0001)
```yml
script:
  - ls
  # 输出结果没有 A
```
 
### 其他参数

列举一些使用上有用的参数，基本见名知意，没有坑点就不展开了，查阅文档即可方便使用。

`allow_failed`
`dependencies`
`retry`
`artifacts`

## 其他非 yml 部分

谈一谈yml之外的，跟本文有有一点点相关的。
通过分层，分仓库的方式，来实现一些定制的功能：

ymls 仓库，作为统一存储 yml 的地方
- 存储 **库统一使用的yml**
- 存储 **脚本库更新本地脚本的的yml**
- 一些简单的日志打点脚本

脚本仓库，存储所有使用的脚本，
- 引用 **更新本地脚本的的yml**
- 存储各个 job 的脚本
- 存储全局黑白名单
  
还有就是咱们的代码仓库了：

代码库
- 使用 **库统一使用的yml**
- 项目代码


### 日志

因为 gitlab 的 ci 没有一个大盘（大佬知道的纠正我），所以需要自己记录。

以前我们是没有记录的，那现在就可以把它写在统一的 yml 配置 的 before 和 after 相关脚本上。

同时在旧脚本上加上日志能作一些对比，还能发现一些问题。

### 关于性能清理

- 关于打包

之前一个库，如果有十个左右的字库，完整的打包需要30分钟以上。但是我们在项目中使用是完整导入的，分子库更多的是为了代码隔离。
现在使用了打包整个库的形式，加上快速发布，基本在5分钟左右即可。

- 关于 gitlab 频繁拉脚本仓库

上面的知识有提到 job 的沙箱特性，所以需要拉脚本。以前每个库发包就需要执行 5次左右的 git clone，新改造之后从本地复制，将在频繁打包时候减少很多连接错误。
本地的更新通过添加脚本仓库的ci，每次更新脚本时候即更新机器上相应目录的脚本。


### 关于全局配置文件

通过全局的黑白名单，进行不同库的定制化脚本即可。

那其实也有个好处，知道当前大家所有库的当前一些选项开关，一个小盘。



## 后续

那通过上面的一些措施，功能上实现了上面的规划：
1. 全局配置问题
2. CI 收归，支持动态更新 job 问题
3. 个性化配置，不同仓库执行不同 job 问题
4. CI日志问题

有些因为 gitlab 提供的能力不够大，如果后续有更好的支持的话，那就能写出更优雅的代码了。本文就结束了，重点还是在 YML 使用和对于当前的需求的一些理解上，大家多多提意见。
