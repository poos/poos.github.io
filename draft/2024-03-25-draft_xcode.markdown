---
layout:     post
title:      
subtitle:   app exit
hide-in-nav: true
date:       2024-03-25
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- Code
---


iOS证书 描述文件 安装 cert mobile .mobileprovision



# 证书安装不了

如果在Finder安装开发证书安装失败，可以尝试如下的备用方式，在命令行工具执行：
security import ~/aaa.keychain

参考自 https://blog.csdn.net/LIUXIAOXIAOBO/article/details/113501374


安装完证书重启Xcode


# 跑低版本或者高版本的iOS

https://www.jianshu.com/p/6eddef3aa779

在Xcode升级到Xcode14以后，大家都发现系统的支持版本升级到了11.0，那么想要调试11.0之前的系统该怎么办呢？
下边给大家介绍Xcode14调试11.0之前的系统的方法：

￼
截屏2022-09-15 09.55.56.png
1.首先在Xcode14之前的版本下，应用程序-xcode 显示包内容 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport ，把程序中的低于11.0的文件夹拷贝到Xcode14的相应目录下。
2.在Xcode14下，应用程序-xcode 显示包内容 修改以下文件,记得修改前备份,防止改错无法还原.
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/SDKSettings.json
SDKSettings.json

{
    "DisplayName": "iOS 16.0",
    "DefaultProperties": {
        "CODE_SIGN_ENTITLEMENTS": "",
        "ENTITLEMENTS_REQUIRED": "YES",
        "KASAN_DEFAULT_CFLAGS": "$(KASAN_CFLAGS_CLASSIC)",
        "KASAN_DEFAULT_CFLAGS[arch=arm64]": "$(KASAN_CFLAGS_TBI)",
        "KASAN_DEFAULT_CFLAGS[arch=arm64e]": "$(KASAN_CFLAGS_TBI)",
        "DEAD_CODE_STRIPPING": "YES",
        "GCC_THUMB_SUPPORT": "YES",
        "TEST_LIBRARY_SEARCH_PATHS": "$(inherited) $(PLATFORM_DIR)\/Developer\/$(TEST_FRAMEWORK_DEVELOPER_VARIANT_SUBPATH)usr\/lib",
        "SUPPORTED_DEVICE_FAMILIES": "1,2",
        "PLATFORM_NAME": "iphoneos",
        "KASAN_CFLAGS_CLASSIC": "-DKASAN=1 -DKASAN_CLASSIC=1 -fsanitize=address -mllvm -asan-globals-live-support -mllvm -asan-force-dynamic-shadow",
        "CODE_SIGNING_REQUIRED": "YES",
        "IPHONEOS_DEPLOYMENT_TARGET": "16.0",
        "TEST_FRAMEWORK_SEARCH_PATHS": "$(inherited) $(PLATFORM_DIR)\/Developer\/$(TEST_FRAMEWORK_DEVELOPER_VARIANT_SUBPATH)Library\/Frameworks $(SDKROOT)\/Developer\/Library\/Frameworks",
        "CODE_SIGN_IDENTITY": "Apple Development",
        "KASAN_CFLAGS_TBI": "-DKASAN=1 -DKASAN_TBI=1 -fsanitize=kernel-hwaddress -mllvm -hwasan-recover=0 -mllvm -hwasan-instrument-atomics=0 -mllvm -hwasan-instrument-stack=1 -mllvm -hwasan-uar-retag-to-zero=1 -mllvm -hwasan-generate-tags-with-calls=1 -mllvm -hwasan-instrument-with-calls=1 -mllvm -hwasan-use-short-granules=0  -mllvm -hwasan-memory-access-callback-prefix=__asan_",
        "ENTITLEMENTS_DESTINATION": "Signature",
        "DEFAULT_COMPILER": "com.apple.compilers.llvm.clang.1_0",
        "DEPLOYMENT_TARGET_SUGGESTED_VALUES": [
            "8.0",
            "8.1",
            "8.2",
            "8.3",
            "8.4",
            "9.0",
            "9.1",
            "9.2",
            "9.3",
            "10.0",
            "10.1",
            "10.2",
            "10.3",
            "11.0",
            "11.1",
            "11.2",
            "11.3",
            "11.4",
            "12.0",
            "12.1",
            "12.2",
            "12.3",
            "12.4",
            "13.0",
            "13.1",
            "13.2",
            "13.3",
            "13.4",
            "13.5",
            "13.6",
            "14.0",
            "14.1",
            "14.2",
            "14.3",
            "14.4",
            "14.5",
            "14.6",
            "14.7",
            "15.0",
            "15.1",
            "15.2",
            "15.3",
            "15.4",
            "15.5",
            "15.6",
            "15.7",
            "16.0",
            "16.1"
        ],
        "AD_HOC_CODE_SIGNING_ALLOWED": "NO"
    },
    "MinimalDisplayName": "16.0",
    "Version": "16.0",
    "IsBaseSDK": "YES",
    "SupportedTargets": {
        "iphoneos": {
            "LLVMTargetTripleVendor": "apple",
            "DeploymentTargetSettingName": "IPHONEOS_DEPLOYMENT_TARGET",
            "SwiftConcurrencyMinimumDeploymentTarget": "15.0",
            "Archs": [
                "arm64e",
                "arm64"
            ],
            "LLVMTargetTripleEnvironment": "",
            "ClangRuntimeLibraryPlatformName": "ios",
            "MaximumDeploymentTarget": "16.0.99",
            "BuildVersionPlatformID": "2",
            "DefaultDeploymentTarget": "16.0",
            "LLVMTargetTripleSys": "ios",
            "DeviceFamilies": [
                {
                    "Identifier": "1",
                    "Name": "phone",
                    "DisplayName": "iPhone"
                },
                {
                    "Identifier": "2",
                    "Name": "pad",
                    "DisplayName": "iPad"
                }
            ],
            "MinimumDeploymentTarget": "8.0",
            "SwiftOSRuntimeMinimumDeploymentTarget": "12.2",
            "RecommendedDeploymentTarget": "12.5",
            "PlatformFamilyName": "iOS",
            "ValidDeploymentTargets": [
                "8.0",
                "8.1",
                "8.2",
                "8.3",
                "8.4",
                "9.0",
                "9.1",
                "9.2",
                "9.3",
                "10.0",
                "10.1",
                "10.2",
                "10.3",
                "11.0",
                "11.1",
                "11.2",
                "11.3",
                "11.4",
                "12.0",
                "12.1",
                "12.2",
                "12.3",
                "12.4",
                "13.0",
                "13.1",
                "13.2",
                "13.3",
                "13.4",
                "13.5",
                "13.6",
                "14.0",
                "14.1",
                "14.2",
                "14.3",
                "14.4",
                "14.5",
                "14.6",
                "14.7",
                "15.0",
                "15.1",
                "15.2",
                "15.3",
                "15.4",
                "15.5",
                "15.6",
                "15.7",
                "16.0",
                "16.1"
            ],
            "SystemPrefix": ""
        }
    },
    "PropertyConditionFallbackNames": [
        "embedded"
    ],
    "DefaultDeploymentTarget": "16.0",
    "MaximumDeploymentTarget": "16.0.99",
    "DebuggerOptions": {
        "SupportsViewDebugging": "YES"
    },
    "CanonicalName": "iphoneos16.0",
    "CustomProperties": {}
}


/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/SDKSettings.plist
添加 8.0-10.3 的版本 到以下节点中
SupportedTargets - iphoneos - ValidDeploymentTargets
DefaultProperties - DEPLOYMENT_TARGET_SUGGESTED_VALUES
如果没有包含，请把这个plist文件拷贝到桌面手动添加。添加完成后再粘贴到原来的位置。重启Xcode即可。
3.MinimumDeploymentTarget 改为8.0
DeviceSupport下载地址:
https://github.com/iGhibli/iOS-DeviceSupport
链接：https://www.jianshu.com/p/1fab8881f08a





-------------



-------------

xcodebuild 编译打包 / scp 上传文件


xco64 \
ENABLE_BITCODE=Disable \
archive -archivePath ipadebuild test -project iOSXCTestApp.xcodeproj -scheme iOSXCTestApp -destination platform="iOS Simulator",name="iPhone SE (2nd generation)" -enablePerformanceTestsDiagnostics YES

xcodebuild test -workspace aaa.xcworkspace -scheme AaaUITests -destination platform=iOS,id=00008030-0010459802D0802E -enablePerformanceTestsDiagnostics YES


xcodebuild -workspace AAa.xcworkspace \
-scheme AAa \
-configuration Debug \
-allowProvisioningUpdates -allowProvisioningDeviceRegistration \
-sdk iphonesimulator \
-UseModernBuildSystem=0 \
LD_GENERATE_MAP_FILE=$(SRCROOT)/$(PRODUCT_NAME)-LinkMap.txt \
LD_GENERATE_MAP_FILE=YES \
EXCLUDED_ARCHS=arm/aaa.xcarchive \
-quiet


pod gen aaa.git --verbose




-------------

CICD gitlab api OCLint

gitlab runner

https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-section

https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html#operator-properties





https://docs.gitlab.com/ee/user/application_security/dependency_scanning/index.html

https://docs.gitlab.com/ee/ci/yaml/index.html#artifactsreportscodequality

https://docs.codeclimate.com/docs/list-of-engines


https://blog.csdn.net/weixin_55437537/article/details/113885455


https://docs.gitlab.com/ee/user/project/merge_requests/code_quality.html




https://gitlab.example.com/api/v4/projects/5/repository/branches




-------------

shell 文件路径

# #!/bin/bash
# function getdir(){
#   for element in `ls $1`
#     do
#       dir_or_file=$1"/"$element
#   if [ -d $dir_or_file ]
#     then
#       getdir $dir_or_file
#     else
#       echo $dir_or_file     
#   fi
# done
# }


ruby 文件路径

  files = Pathname.glob("aa/**/*.{h,m,mm}")
  files = files.map {|file| file.to_path}




if [ ! "~/Desktop/aaa.ipa" ]; then
    echo "a"
fi
echo b





-------------

xcrun altool --validate-app -f /Users/aaa/a.ipa -u useremail -p app-token-app-token -t ios  --show-progress --verbose 

xcrun altool --upload-app -f /Users/aaa/a.ipa -u useremail -p app-token-app-token -t ios  --show-progress --verbose 




-------------


print isa


void printIsa(id obj){
    struct IsaObjc {
        long isa;
    };
    long p = ((__bridge struct IsaObjc *)obj) -> isa;
    int endFlag = 45;
#if __x86_64__
    endFlag = 56;
#endif
    
    NSLog(@"break----> isa: 0x%lx  \n nonpointer: %ld \n has_assoc: %ld \n has_cxx_dtor: %ld \n\
          shiftcls: 0x%lx \n magic: 0x%lx \n weakly_referenced: %ld \n deallocating: %ld \n has_sidetable_rc: %ld \n extra_rc: %lu ",              // isa存储引用count，满了后移动一部分到SideTables，自身保留2^18(mac 2^7), 保证性能。
          p,
          p & 0x1,
          (p & 0x2) >> 1,
          (p & 0x4) >> 2,
          (p & 0x00007ffffffffff8),
          (p & 0x001f800000000001) >> (endFlag - 6),
          (p & 0x0020000000000000) >> (endFlag - 3),
          (p & 0x0040000000000000) >> (endFlag - 2),
          (p & 0x0080000000000000) >> (endFlag - 1),
          (p & 0xff00000000000000) >> endFlag);
}




-------------
模拟器


Edit > Automatically Sync Pasteboard





-------------

https://mp.weixin.qq.com/s/CiqMlEIp1Ir2EJSDGgMooQ




http://t.zoukankan.com/mukekeheart-p-8144742.html


http://t.zoukankan.com/mukekeheart-p-8144742.html


sigkill





-------------


LD_GENERATE_MAP_FILE


LD_GENERATE_MAP_FILE=$(SRCROOT)/$(PRODUCT_NAME)-LinkMap.txt \
LD_GENERATE_MAP_FILE=YES \


-------------

忽略警告


#pragma clang diagnostic push
#pragma clang diagnostic ignored "警告名称"
 
// 被夹在这中间的代码针对于此警告都会忽视不显示出来
 
//常见警告的名称
//1.声明变量未使用  "-Wunused-variable"
//2.方法定义未实现  "-Wincomplete-implementation"
//3.未声明的选择器  "-Wundeclared-selector"
//4.参数格式不匹配  "-Wformat"
//5.废弃掉的方法     "-Wdeprecated-declarations"
//6.不会执行的代码  "-Wunreachable-code"
//7.忽略在arc 环境下performSelector产生的 leaks 的警告 "-Warc-performSelector-leaks"
//8.忽略类别方法覆盖的警告 "-Wobjc-protocol-method-implementation"（修复开源库bug，覆盖开源库方法时会用到）
 
#pragma clang diagnostic pop




-------------

Git——Git 停止跟踪文件


1 人赞同了该文章
在开发仓库中，我们经常遇到这样的情况，我们不再希望在 Git 中跟踪某些文件的更改。
假设我们有一个文件，我们现在觉得它是多余的，不再与项目相关。在这种情况下，我们希望从 Git 仓库中的跟踪中删除该文件。
我们现在将通过一个例子来说明这一点。
使用 git rm 停止跟踪 Git 中的文件
假设我们在 Git 的仓库中有一个名为 file1 的文件，我们不再希望跟踪它。
我们可以使用带有 --cached 选项的 git rm 命令从 Git 中的跟踪中删除该文件。
$ git rm --cached file1
rm 'file1'
我们还可以使用以下命令从 Git 仓库中的跟踪中删除文件夹。

$ git rm -r --cached <folder-name>
这将从跟踪中删除指定的文件或文件夹（即）从索引中删除它；但不会从文件系统中删除该文件。
注意
注意：当我们在其他机器上执行 git pull 以从远程仓库获取新更改时，该文件或文件夹将从该文件系统中删除。当我们从远程仓库新克隆时，这也会导致删除文件或文件夹。
另请注意，我们需要提交文件删除以更新远程仓库中的此更改。




-------------
appledoc \
--output ./apiDoc \
-i *.m \
-m \
--project-name "AaAa" \
--project-company "AAA" \
--no-create-docset \
--keep-undocumented-objects \
--keep-undocumented-members \
--no-warn-undocumented-object \
--no-warn-undocumented-member  \
./








-------------



shell 脚本包含字符串


假设我们定义了一个变量为：
file=/dir1/dir2/dir3/my.file.txt
可以用${ }分别替换得到不同的值：
${file#*/}：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/}：删掉最后一个 /  及其左边的字符串：my.file.txt
${file#*.}：删掉第一个 .  及其左边的字符串：file.txt
${file##*.}：删掉最后一个 .  及其左边的字符串：txt
${file%/*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3
${file%%/*}：删掉第一个 /  及其右边的字符串：(空值)
${file%.*}：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*}：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my

记忆的方法为：
# 是 去掉左边（键盘上#在 $ 的左边）
%是去掉右边（键盘上% 在$ 的右边）
单一符号是最小匹配；两个符号是最大匹配
${file:0:5}：提取最左边的 5 个字节：/dir1
${file:5:5}：提取第 5 个字节右边的连续5个字节：/dir2
也可以对变量值里的字符串作替换：
${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
${file//dir/path}：将全部dir 替换为 path：/path1/path2/path3/my.file.txt

利用 ${ } 还可针对不同的变数状态赋值(沒设定、空值、非空值)： 
${file-my.file.txt} ：假如 $file 沒有设定，則使用 my.file.txt 作传回值。(空值及非空值時不作处理) 
${file:-my.file.txt} ：假如 $file 沒有設定或為空值，則使用 my.file.txt 作傳回值。 (非空值時不作处理)
${file+my.file.txt} ：假如 $file 設為空值或非空值，均使用 my.file.txt 作傳回值。(沒設定時不作处理)
${file:+my.file.txt} ：若 $file 為非空值，則使用 my.file.txt 作傳回值。 (沒設定及空值時不作处理)
${file=my.file.txt} ：若 $file 沒設定，則使用 my.file.txt 作傳回值，同時將 $file 賦值為 my.file.txt 。 (空值及非空值時不作处理)
${file:=my.file.txt} ：若 $file 沒設定或為空值，則使用 my.file.txt 作傳回值，同時將 $file 賦值為 my.file.txt 。 (非空值時不作处理)
${file?my.file.txt} ：若 $file 沒設定，則將 my.file.txt 輸出至 STDERR。 (空值及非空值時不作处理)

${file:?my.file.txt} ：若 $file 没设定或为空值，则将 my.file.txt 输出至 STDERR。 (非空值時不作处理)
${#var} 可计算出变量值的长度：

${#file} 可得到 27 ，因为/dir1/dir2/dir3/my.file.txt 是27个字节


方法二：利用字符串运算符


strA="helloworld"
strB="low"
if [[ $strA =~ $strB ]]
then
    echo "包含"
else
    echo "不包含"
fi


利用字符串运算符 =~ 直接判断strA是否包含strB。（这不是比第一个方法还要简洁吗摔！）
 
方法三：利用通配符


A="helloworld"
B="low"
if [[ $A == *$B* ]]
then
    echo "包含"
else
    echo "不包含"
fi


这个也很easy，用通配符*号代理strA中非strB的部分，如果结果相等说明包含，反之不包含。
 


-------------

shell 只保留几个远程tag

sync_tags() {
    git tag | xargs git tag -d
    git fetch $1 -p
}

# git只保留最新的N个tag
N=15
# git push --tags
sync_tags origin
tags_count=$(git tag | wc -l | awk '{print $1}')
# tags按照时间排序： http://stackoverflow.com/questions/6269927/how-can-i-list-all-tags-in-my-git-repository-by-the-date-they-were-created
if [ $tags_count -gt $N ]; then
    git for-each-ref --sort=taggerdate --format '%(refname:short)' refs/tags | awk -v n=$N '{l[NR]=$0} END {for (i=1; i<=NR-n; i++) print l[i]}' | xargs -n 1 git push origin --delete
    sync_tags origin
else
    echo "only $tags_count tags"
fi




-------------


Linux shell脚本重试机制

重试机制在实际编程场景中应用比较场景，比如你的任务在请求一个正在写入数据但不确定什么时间会完成的文件，可能就需要通过尝试机制间隔一段时间重新执行任务。
以下 shell 脚本是实现重试机制的模板：
￼
#!/bin/sh

count=0     #记录重试次数
flag=0      # 重试标识，flag=0 表示任务正常，flag=1 表示需要进行重试
while [ 0 -eq 0 ]
do
    echo ".................. job begin  ..................."
    # ...... 添加要执行的内容，flag 的值在这个逻辑中更改为1，或者不变......
    
    # 检查和重试过程   
    if [ flag -eq 0 ]; then     #执行成功，不重试
        echo "--------------- job complete ---------------"
        break;
    else                        #执行失败，重试
        count=$[${count}+1]
        if [ ${count} -eq 6 ]; then     #指定重试次数，重试超过5次即失败
            echo 'timeout,exit.'
            break
        fi
        echo "...............retry in 2 seconds .........."
        sleep 2
    fi
done
￼


#!/bin/sh -xe -x表示打印执行的每个命令。 -e表示如果脚本中的任何命令失败，则退出失败。
#--------------

#!/bin/sh -x


job1() {
    echo "job 1"
   
}

job2() {
    echo "job 2"
    
}

echo "========start job 1========"
echo "========times ${SECONDS}s========"
job1_max_times=3
job1_times=1
while [ 0 -eq 0 ]; do
    echo "-----try ${job1_times}-----"
    job1
    if [ $? -eq 0 ]; then
        echo "========times ${job1_times} success ${SECONDS}s========"
        break
    else

        if [ $job1_times -ge $job1_max_times ]; then
            echo "times ${job1_times} failed, exit"
            exit 1
            break

        else
            job1_times=$(($job1_times + 1))
            echo "-----times ${job1_times} failed, try after 10s-----"
            sleep 10
        fi
    fi
done
echo "========done times ${SECONDS}s========"

echo "========start job 2========"
echo "========times ${SECONDS}s========"
job2_max_times=3
job2_times=1
while [ 0 -eq 0 ]; do
    echo "-----try ${job2_times}-----"
    job2
    if [ $? -eq 0 ]; then
        echo "========times ${job2_times} success ${SECONDS}s========"
        break
    else

        if [ $job2_times -ge $job2_max_times ]; then
            echo "times ${job2_times} failed，exit"
            exit 1
            break

        else
            job2_times=$(($job2_times + 1))
            echo "-----try ${job2_times} failed, try after 10s-----"
            sleep 10
        fi
    fi
done
echo "========done ${SECONDS}s========"



-------------



1、打开终端APP2、输入:cd 输入framework文件所在目录(包含framework文件)/也可以直接拖入framework文件之后回车3、输入:file framework文件夹内二进制文件如果输出结果包含dynamically linked shared library则是动态库，反之则为静态库。



作者：清宵寒夜
链接：https://www.jianshu.com/p/630bb96fecd0
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



-------------


TEMP


COCOAPODS_DISABLE_DETERMINISTIC_UUIDS=true pod install



-------------



swiftlint

md

现有的规则 和 正则规则

自定义规则

swiftc -dump-parse ViewController2.swift
sourcekitten structure --file ViewController2.swift

oc

xcode ide



-------------


静态图的标签用的是腾讯的 Jietu，MAS 上免费下载。
动图的击键提示是开源的 KeyCastr。



-------------

iOS模拟器发生了崩溃，可以在如下地方找到崩溃日志：
~/Library/Logs/DiagnosticReports/




-------------


小知识点

每次看到人家的十六进制输出，对齐的很好，ff就显示了，而我的总是0xffffffff。
如果是"%02x"，是以0补齐2位数，如果超过2位就显示实际的数；
"%hhx" 是只输出2位数，即便超了，也只显示低两位；
因此有了"%02hhx"




-------------

oc lint

#!/bin/bash

# 指定编码
export LANG="zh_CN.UTF-8"
export LC_COLLATE="zh_CN.UTF-8"
export LC_CTYPE="zh_CN.UTF-8"
export LC_MESSAGES="zh_CN.UTF-8"
export LC_MONETARY="zh_CN.UTF-8"
export LC_NUMERIC="zh_CN.UTF-8"
export LC_TIME="zh_CN.UTF-8"
export LC_ALL=

FORMAT_MODULE=
# 项目名称
CurrentDir="$( pwd )"
echo $CurrentDir
cd Example

echo "-----执行pod bin install-----"
pod bin install

echo "-----检查xcworkspace和scheme-----"
project_name=`find . -name *.xcodeproj | awk -F "[/.]" '{print $(NF-1)}'`
project_path='./'
if [ -e "${project_path}/${project_name}.xcworkspace" ];then
    workspace_name="${project_name}.xcworkspace"
    echo "工程名：${workspace_name}"
else
    echo "no xcworkspace"
    exit 1
fi
myworkspace=${workspace_name}
myscheme=`xcodebuild -list -json | awk '$0~/scheme/{getline; print $0;}' | awk -F "[\"]" '{print $2}'`
echo "-----myworkspace是：${myworkspace}, myscheme是：${myscheme}-----"

# 清除上次编译数据
echo '-----执行xcodebuild clean-----'
xcodebuild -workspace ${myworkspace} -scheme ${myscheme} clean

echo '-----开始编译-----'
# 生成编译数据
xcodebuild \
ARCHS=x86_64 \
ONLY_ACTIVE_ARCH=NO \
COMPILER_INDEX_STORE_ENABLE=NO \
-workspace ${myworkspace} -scheme ${myscheme} \
-sdk iphonesimulator \
-configuration Debug \
 | xcpretty -r json-compilation-database -o compile_commands.json

if [ -f ./compile_commands.json ]
    then
    echo "-----编译数据生成完毕-----"
else
    echo "-----生成编译数据失败-----"
    exit 2
fi

echo '-----分析中-----'
oclint-json-compilation-database -e Pods -- -report-type html -o oclintReport.html

rm compile_commands.json
echo '-----分析完毕-----'
exit 0




-------------

oclint


  550  brew list
  551  oclint
  552  brew tap oclint/formulae
  553  vi .bash_profile
  554  vi .zshrc
  555  brew install oclint
  556  brew --help
  557  brew hel[ tap
  558  brew help tap
  559  oclint
  560  ./aaa.sh "Pods/HCBKVO"
  561  ls
  562  ls -gal
  563  ls
  564  ls -gal
  565  cd /bin
  566  ls
  567  cd /
  568  ls
  569  ls -gal
  570  cd bin
  571  ls
  572  oclint
  573  /usr/local/Cellar/oclint/20.11
  574  ls
  575  cd bin
  576  ls
  577  oclint
  578  oclint-20.11
  579  ./oclint
  580  open .bash_profile
  581  source .bash_profile
  582  oclint
  583  open .zshrc
  584  source .zshrc
  585  oclint
  586  source .zshrc
  587  oclint
  588  brew uninstall oclint
  589  brew tap oclint/formulae\nbrew install oclint
  590  ./oclint
  591  oclint
  592  echo &PATH
  593  echo $PATH
  594  cd /usr/local/Cellar/oclint/20.11
  595  cd bin
  596  pwdd
  597  pwd
  598  ls
  599  .oclint
  600  .oclint-20.11
  601  ./oclint-20.11
  602  ./oclint
  603  vi bash_podfole
  604  pwd
  605  source .zshrc
  606  oclint
  607  ls
  608  cd mbrepos/aaa
  609  ls
  610  ./oclint_apply.sh
  611  vi compile_commands.json
  612  ls
  613  vi compile_commands.json
  614  ls
  615  ./oclint_apply.sh
  616  cat compile_commands.json
  617  ls
  618  ./oclint_apply.sh
  619  vi compile_commands.json
  620  ./oclint_apply.sh
  621  cat compile_commands.json
  622  ./oclint_apply.sh
  623  cat compile_commands.json
  624  ls
  625  pwd
  626  cd .
  627  cd ..
  628  ls
  629  open .
  630  ./oclint_apply.sh
  631  ls -gal
  632  cat compile_commands.json
  633  cd ..
  634  git clone git@code.aaa.com:iOSaaa.git
  635  cd aaa
  636  ls
  638  cd -
  639  pod install
  640  cd mbrepos
  641  git@code.aaa.com:iOS-Team/aaa.git
  642  git clone git@code.aaa.com:iOS-Team/aaa.git
  643  ls
  644  pod install
  645  cd .scripts
  646  ls
  647  open aaa.sh
  648  pod update
  649  pod install
  650  cd ..
  651  pod update
  652  cd aaa
  653  ls
  654  ./oclint_apply.sh
  655  ls -gla
  656  cat compile_commands.json
  657  pod update
  658  ls
  659  ./aaa.sh
  660  ./oclint_apply.sh
  661  oclint
  662  cd mbrepos
  663  cd aaa
  664  ls
  665  ./aaa.sh
  666  oclint
  667  brew install swiftlint
  668  swiftlint
  669  ls
  670  pod bin install
  671  ls
  672  cd .scripts
  673  cd ..
  674  ./aaa.sh
  675  cd mbrepos
  676  git clone git@code.aaa.com:iOS-Team/aaa.git
  677  cd HCBKVO/Example
  678  ls
  679  open .
  680  vi aaa.sh
  681  ./aaa.sh
  682  chmod +x aaa.sh
  683  ./aaa.sh
  684  open aaa.sh
  685  pod bin install
  686  ./aaa.sh
  687  open .
  688  xcodebuild list
  689  xcodebuild -list
  690  xcodebuild -target HCBKVO_Example -configuration Debug -scheme HCBKVO-Example
  691  cd mbrepos/MBFoundation
  692  cd Example
  693  open .
  694  xcodebuild
  695  xcodebuild -list
  696  xcodebuild -target MBFoundation_Example -configuation Debug -scheme MBFoundation-Example
  697  xcodebuild -target MBFoundation_Example -configuration Debug -scheme MBFoundation-Example
  698  oclint-xcodebuild
  699  xcodebuild -target MBFoundation_Example -configuration Debug -scheme MBFoundation-Example
  700  xcodebuild -target HCBKVO_Example -configuration Debug -scheme HCBKVO-Example
  701  xcodebuild -target MBFoundation_Example -configuration Debug -scheme MBFoundation-Example
  702  pwd
  703  xcodebuild -target MBFoundation_Example -configuration Debug -scheme MBFoundation-Example
  704  xcodebuild 
  705  xcpretty
  706  ls
  707  vi codereview.sh
  708  chmod -x codereview.sh
  709  open .
  710  ./codereview.sh
  711  chmod +x codereview.sh
  712  ./codereview.sh
  713  open .
  714  ./codereview.sh
  715  ls
  716  ./aaa.sh
  717  open .
  718  ./aaa.sh
  719  oclint-json-compilation-database
  720  ./aaa.sh
  721  pwd
  722  ./aaa.sh
  723  pwd
  724  ./aaa.sh
  725  cd /Users/bieshixuan/aaa.
  726  ./aaa.sh
  727  oclint-json-compilation-database --helper
  728  ./aaa.sh
  729  pwd
  730  open .
  731  ./aaa.sh
  732  open .
  733  ./aaa.sh
  734  /usr/local/Cellar/oclint/20.11/bin/oclint --help\n
  735  oclint-json-compilation-database --help
  736  ./aaa.sh
  737  oclint-json-compilation-database -e Pods -e ManagerProject/Tools -- -rc=LONG_LINE=300 -disable-rule=LongLine -report-type html -o=report.html -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
  738  oclint-json-compilation-database oclint_args -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
  739  oclint-json-compilation-database ./$myscheme -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
  740  oclint-json-compilation-database -i Pods -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999\n
  741  /usr/local/Cellar/oclint/20.11/bin/oclint --help\n
  742  oclint-json-compilation-database
  743  oclint-json-compilation-database --help
  744  oclint-json-compilation-database -i Pods oclint_args / -- -report-type html -o oclintReport.html -rc LONG_LINE=9999 -max-priority-1=9999 -max-priority-2=9999 -max-priority-3=9999
  745  oclint-json-compilation-database -i Pods .oclint / -- -report-type html -o oclintReport.html
  746  oclint-json-compilation-database -i Pods .oclint -- -report-type html -o oclintReport.html
  747  oclint-json-compilation-database -i Pods .oclint -- -report-type html -o oclintReport.htmloclint-json-compilation-database .oclint -i Pods -- -report-type html -o oclintReport.html
  748  oclint-json-compilation-database .oclint -i Pods -- -report-type html -o oclintReport.html
  749  oclint-json-compilation-database -v -i Pods .oclint -- -report-type html -o oclintReport.html
  750  oclint-json-compilation-database --help
  751  open .
  752  pwdd
  753  pwd
  754  ./aaa.sh
  755  oclint-json-compilation-database -v -e Pods -i Development\ Pods .oclint -- -report-type html -o oclintReport.html
  756  ./aaa.sh
  757  oclint-json-compilation-database -v -e Pods -i Development\ Pods -- .. -report-type html -o oclintReport.html
  758* oclint --help
  759  oclint-json-compilation-database -v -e Pods -i Development\ Pods -- build -report-type html -o oclintReport.html
  760  oclint-json-compilation-database -v -e Pods -i Development\ Pods -- /build -report-type html -o oclintReport.html
  761  oclint-json-compilation-database -v -e Pods -i Development\ Pods -- build -report-type html -o oclintReport.html
  762* cd /Users/bieshixuan/mbrepos/HCBKVO/Example/build
  763* ls
  764  oclint-json-compilation-database -v -e Pods -i Development\ Pods -- build -report-type html -o oclintReport.html
  765  xcodebuild | xcpretty -r json-compilation-database
  766  \nxcodebuild ARCHS=x86_64 \\nONLY_ACTIVE_ARCH=NO \\n#OBJROOT="${OBJROOT}/DependentBuilds" \\nCOMPILER_INDEX_STORE_ENABLE=NO \\n-workspace ${myworkspace} -scheme ${myscheme} \\n-sdk iphonesimulator \\n-configuration Debug \\n| xcpretty -r json-compilation-database -o compile_commands.json
  767  ./aaa.sh
  768  oclint-json-compilation-database
  769  ./aaa.sh

