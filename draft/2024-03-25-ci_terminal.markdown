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
# 做CI打包的那些岁月 - 命令行

## 历史

```
: 1708568827:0;git tag 0.2.44 -m "aaa"
: 1708568828:0;sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
: 1708568829:0;find . | grep "XX/"
: 1708568830:0;find . | grep "XX/" | xargs rm -rf
: 1708568831:0;scp user@10.191.168.1:/Users/user/Desktop/aa.txt ~/Desktop
: 1708568832:0;xcodebuild -workspace AA.xcworkspace -scheme AA -configuration Debug -sdk iphonesimulator
: 1708568833:0;xcodebuild -workspace aa.xcworkspace -scheme aa -configuration Debug -sdk iphoneos | xcpretty
: 1708568834:0;python3 --version
: 1708568835:0;pip3 install python-gitlab
: 1708568836:0;swift aa.swift /Users/aa/LinkMap.txt
: 1708568837:0;brew install ideviceinstaller
: 1708568838:0;ideviceinstaller -l
: 1708568839:0;idevicesyslog
: 1708568840:0;sudo gem install rouge -v 3.30.0
: 1708568841:0;sudo gem install jekyll#sudo gem install jekyll -v 3.2#ruby2.6de
: 1708568842:0;bundle install;jekyll serve -w#bundle exec jekyll serve -w
: 1708568843:0;open http://127.0.0.1:4000
: 1708568844:0;sudo gem install bundler -v 2.0.2
: 1708568845:0;bundle update
: 1708568846:0;md5 /a.framework/a
: 1708568847:0;touch inject_variables_txt.txt
: 1708568848:0;cat inject_variables_txt.txt
: 1708568849:0;xcrun altool --upload-app --help
: 1708568850:0;xcrun altool --upload-app -f ${ipa_outpath}/${target}.ipa -u ping.xiang@team.com -p meoz-keuk-eftb-ebsa -t ios
: 1708568851:0;ps -fe | grep job2
: 1708568852:0;ps -fe | grep asdqx_job2 | grep -v grep
: 1708568853:0;ps -fe | grep asdqx_job2 | awk '{print $2}'
: 1708568854:0;ps -fe | grep random_asdqx_job2 | awk '{print $2}'
: 1708568855:0;kill 14167
: 1708568856:0;perl --help
: 1708568857:0;perl -e 'print "Hello World\n"'
: 1708568858:0;perl -e 'alarm shift; exec @ARGV'
: 1708568859:0;perl -e 'exec @ARGV' pod update
: 1708568860:0;alram shift 3; echo "done"
: 1708568861:0;alarm shift 3; echo "done"
: 1708568862:0;perl -e 'alarm shift 3; echo d'
: 1708568863:0;perl -e 'alarm shift; 3 echo d'
: 1708568864:0;perl -e 'alarm shift;' 3
: 1708568865:0;perl -e 'alarm shift;' 3 echo 3
: 1708568866:0;perl -e 'alarm shift; @ARGV' 3 echo 3
: 1708568867:0;perl -e 'alarm shift; echo @ARGV' 3 echo 3
: 1708568868:0;perl -e 'alarm shift; echo @ARGV' 3
: 1708568869:0;exec
: 1708568870:0;exec -h
: 1708568871:0;exec --help
: 1708568872:0;alarm shift
: 1708568873:0;perl -e 'alarm shift'
: 1708568874:0;perl -e 'alarm shift;'
: 1708568875:0;perl -e 'alarm shift 3'
: 1708568876:0;perl -e 'alarm shift;3'
: 1708568877:0;perl -e 'alarm shift;3;echo 1'
: 1708568878:0;perl -e 'alarm shift;3' echo 1
: 1708568879:0;curl -I -m 10 -o /dev/null -s -w %{http_code}  www.baidu.com
: 1708568880:0;gem list cocopods
: 1708568881:0;gem search cocoapods-bin
: 1708568882:0;ping aaa.com
: 1708568883:0;pod bin archive --clean --sources=git@code.team.com:iOS/specrepo.git,git@code.team.com:cocoapods/thirdpartyspecrepo.git --verbose --use-modular-headers --create-umbrella-header  --configuration=Release
: 1708568884:0;whtch gem
: 1708568885:0;where gem
: 1708568886:0;git  --no-pager diff Podfile
: 1708568887:0;pod gen UIKit.podspec --sources=git@code.team.com:iOS/specrepo.git,git@code.team.com:cocoapods/thirdpartyspecrepo.git --verbose --platforms=ios --use-podfile --podfile-path=/Users/user/repos/UIKit/Example/Podfile
: 1708568888:0;atos -o Desktop/aa.app.dSYM/Contents/Resources/DWARF/aa -arch arm64 -l 1
: 1708568889:0;brew search jq
: 1708568890:0;brew install jq
: 1708568891:0;json_str=$(curl https://aa.com/text)
: 1708568892:0;echo $json_str
: 1708568893:0;touch test.json
: 1708568894:0;echo $json_str > test.json
: 1708568895:0;jq '.message' test.json
: 1708568896:0;/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash --dsym /Users/aa/Desktop/aa.app.dSYM /Users/aa/Downloads/aa.crash
: 1708568897:0;file /Users/user/Desktop/testEnum/testEnum/EnumA.framework/EnumA
: 1708568898:0;curl -H 'Host: image.aa.com' -H 'user-agent: aa/7.27.0 (iPhone; iOS 15.2; Scale/2.00)' -H 'accept: image/*,*/*;q=0.8' -H 'accept-language: zh-CN,zh-Hans;q=0.9' --compressed 'https://image.aa.com/boss/2019/0921/1569046464' --trace output.txt -L -v --trace-time --output output2.txt -L -v
: 1708568899:0;brew install --cask soulver
: 1708568900:0;cd /usr/local/bin && sudo ln -s /Applications/Soulver\ 3.app/Contents/MacOS/CLI/soulver
: 1708568901:0;soulver 0x10+0x10
: 1708568902:0;sed -Ealnru -i "" "s/pod *.KVOController. *\(,[^,]*git[^,]*[\'\"] *\)\?/pod \'KVOController\', \'1.2.0\'/" Podfile
: 1708568903:0;security unlock-keychain -p $PASSWORD ~/Library/Keychains/login.keychain
: 1708568904:0;vmmap -summary iOSXCTestApp.memgraph
: 1708568905:0;heap iOSXCTestApp.memgraph -sortBySize
: 1708568906:0;heap App.memgraph -addresses all
: 1708568907:0;heap App.memgraph -addresses all | _NSZombie_CFString
: 1708568908:0;heap iOSXCTestApp.memgraph -addresses all | "_NSZombie_CFString"
: 1708568909:0;heap --sortBySize iOSXCTestApp.memgraph
: 1708568910:0;malloc_history App.memgraph 0x11fd064c0
: 1708568911:0;leaks App.memgraph
: 1708568912:0;ssh -T ssh://git@git.aa.io:10022
: 1708568913:0;brew install xcbeautify
: 1708568914:0;build ... | xcpretty
: 1708568915:0;fastlane sigh resign IOS_SHIPPER_TF_5.97.2.ipa --signing_identity "Apple Distribution: C Co.,Ltd (77JS533NTY)" -p com.app="build/pp/77JS533NTY/app/AdHoc_com.app.mobileprovision" -p com.app.cargodynamicpush="build/pp/77JS533NTY/app/AdHoc_com.app.cargodynamicpush.mobileprovision"
: 1708568916:0;fastlane run get_provisioning_profile app_identifier:com.app api_key_path:./auth_key/AuthKey_77JS533NTY.json team_id:77JS533NTY output_path:~/Desktop/app skip_certificate_verification:false 'provisioning_name:com.app Release' filename:Release_com.app.mobileprovision cert_id:2F992J3526 --verbose
: 1708568917:0;fastlane sigh download_all --adhoc true --skip_install true -u clewat@foxmail.com -o test4/ -z true
: 1708568918:0;fastlane run update_project_provisioning xcodeproj:./aaa.xcodeproj profile:../iosbuild/build/pp/77JS533NTY/AAaa/AdHoc_com.amh.AAaa.77JS533NTY.mobileprovision target_filter:AAaa build_configuration:AdHoc code_signing_identity:"Apple Distribution: C Co.,Ltd (77JS533NTY)"
: 1708568919:0;fastlane run update_code_signing_settings path:"aaa.xcodeproj" use_automatic_signing:false team_id:"77JS533NTY" targets:"AAaa" build_configurations:"AdHoc" code_sign_identity:"iPhone Distribution: C Co.,Ltd" profile_name:"match AdHoc com.xiwei.yunmanman" bundle_identifier:"com.xiwei.yunmanman2"
: 1708568920:0;fastlane run register_device platform:mac name:xx udid:aaa
: 1708568921:0;fastlane run upload_to_testflight api_key_path:./auth_key/AuthKey_77JS533NTY.json ipa:/Users/user/Desktop/AAaa.ipa
: 1708568922:0;fastlane run create_app_online app_identifier:"com.56qq.GasStationBiz.33NTY77JS5" team_id:"33NTY77JS5" enable_services:"(push_notification:on)"
: 1708568923:0;fastlane ios create33
: 1708568924:0;fastlane produce associate_group -a com.wlqq.driver.liveactivity group.com.wlqq.driver.notifiextension -u user@team.com -b 77JS533NTY
: 1708568925:0;fastlane sigh manage -e
: 1708568926:0;date '+%s'
: 1708568927:0;date '+%m%S'
: 1708568928:0;echo $RANDOM
: 1708568929:0;expr ${RANDOM} + 10000
: 1708568930:0;cat package.json | head -c 5
: 1708568931:0;cat package.json | head -n 5
: 1708568932:0;asfda31413
: 1708568933:0;ls -FR | grep /$ | sed "s:^:`pwd`/:"
: 1708568934:0;ls -R |awk '{print i$0}' i=`pwd`'/'
: 1708568935:0;zip -r Pods/ .
: 1708568936:0;unzip Pods.zip
: 1708568937:0;git reset --hard origin/master
: 1708568938:0;dwarfdump /Users/admin/Library/Developer/Xcode/DerivedData/Ass-cfadweotykjarzdbwenajvkffgbk/Build/Products/Debug-iphoneos/Ass/Ass.a  | grep "AssUtil"
: 1708568939:0;curl -X POST -F token=176e50b751a387e55bbef6100a6d84 -F "ref=dev/0922" -F "variables[COMPONENT_NAME]=AaLib" -F "variables[COMPONENT_VERSION]=0.3.0.4-beta3" -F "variables[COMPONENT_TRIGGERER]=aaa@aap.com" -F "variables[a_TRIGGER_TYPE]=update_tag" http://aa.gitlab.com/api/v4/projects/18/trigger/pipeline
: 1708568940:0;print '\u7cfb\u7edf\u5f02\u5e38\uff0c\u539f\u56e0\u5982\u4e0b\uff1a\u6ca1\u6709\u66f4\u65b0\u6210\u529f'
: 1708568941:0;pod bin archive --clean --sources=git@aa.com:aa/aa.git,git@aa.aa.com:a/a.git --verbose --use-modular-headers --create-umbrella-header --not-provide-swift-module --verbose
: 1708568942:0;pod bin spec create aa.podspec --binary-file-name=aa
: 1708568943:0;pod bin update aaLib
: 1708568944:0;pod bin aaLib
: 1708568945:0;pod cache clean aaLib
: 1708568946:0;sed -i "" "s# *s.source.*#s.source={:http=>"""${aa_oss_url}"""}#" aaa.podspec
: 1708568947:0;security import aa_Dev.p12 -k ~/Library/Keychains/login.keychain
: 1708568948:0;brew install --casks keka
: 1708568949:0;brew search hashcat
: 1708568950:0;brew search --cask betterdisplay
: 1708568951:0;1=/aaa.mobileprovision
: 1708568952:0;eval echo $(egrep -aw -A 1 ExpirationDate $1 | grep date | sed -e 's/<date>//' -e 's/<\/date>//')
: 1708568953:0;date -v "+1m" +%F
: 1708568954:0;echo 20230525 | grep "^\d*$" # not support echo 20230525 | grep "^\d{7}*$"
: 1708568955:0;crumb=$(curl -u "aa@aa:Pwd" http://aa.cn:8080/jenkins/crumbIssuer/api/xml | xmllint --format --xpath 'string(//crumb)' -)
: 1708568956:0;curl -sX POST http://aa.cn:8080/jenkins/job/aa/aa -u "aa@aa.com:61c470002dc73b7a9c9a1071530020e9" -H a7ea1a8fa9c062a834cac4384a37d6f46193689b0e9cbd7ecd563ee2d1e9be2a -d repo_name=aaa
: 1708568957:0;info aaa.framework/Versions/A/aa
: 1708568958:0;lipo -info aaa.framework/Versions/A/aa
: 1708568959:0;arch -x86_64 bash -x a.sh
: 1708568960:0;dyld_info -exports aa.app/aa | wc -l
: 1708568961:0;chmod +x WBBlades
: 1708568962:0;./WBBlades aa.framework
: 1708568963:0;./WBBlades -size
: 1708568964:0;./WBBlades -symbol aa.app -logPath aa.ips
: 1708568965:0;networksetup -listallhardwareports
: 1708568966:0;curl -O http://aa.com:8088/gems/aa.gem
: 1708568967:0;sudo defaults write /Library/Preferences/com.apple.windowserver.plist DisplayResolutionEnabled -bool true
: 1708568968:0;xcode-select -p
: 1708568969:0;xcodebuild -runFirstLaunch
: 1708568970:0;xcrun simctl runtime add /Users/admin/Desktop/iOS_17.2_Simulator_Runtime.dmg
: 1708568971:0;/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
: 1708568972:0;brew install --cask appium
: 1708568973:0;brew install --cask slite
: 1708568974:0;brew install --cask v2rayu
: 1708568975:0;brew install --cask wechat
: 1708568976:0;gem list
: 1708568977:0;brew install --cask appium
: 1708568978:0;brew install --cask youdao
: 1708568979:0;brew install --cask youdaodic
: 1708568980:0;brew install --cask youdaodict
: 1708568981:0;brew install --cask visual-studio-code
: 1708568982:0;sudo gem install cocoapods -v 1.10.1
: 1708568983:0;sudo gem install cocoapods-bin -l aa.gem
: 1708568984:0;sudo gem install aws-sdk-s3
: 1708568985:0;gem sources -a https://gems.ruby-china.com/
: 1708568986:0;gem sources -r https://rubygems.org/
: 1708568987:0;sudo gem install fastlane
: 1708568988:0;gem sources -a https://rubygems.org/
: 1708568989:0;gem sources -r https://gems.ruby-china.com/
: 1708568990:0;gem source -l
: 1708568991:0;sudo curl --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-arm64"
: 1708568992:0;gitlab-runner install
: 1708568993:0;gitlab-runner register
: 1708568994:0;gitlab-runner start
: 1708568995:0;gitlab-runner stop
: 1708568996:0;pip3 install aliyunsdkcore
: 1708568997:0;pip3 install aliyunsdksts
: 1708568998:0;pip3 install aliyun-python-sdk-sts
: 1708568999:0;pip3 install requests
: 1708569000:0;pip3 install oss2
: 1708569001:0;pip3 install python-gitlab
: 1708569002:0;pip3 install mysql-connector -i  http://pypi.douban.com/simple
: 1708569003:0;pip3 install pymongo
: 1708569004:0;cd /etc/ssh; sudo ssh-keygen -t ed25519 -C "user"
: 1708569005:0;df -h           # 命令查看整个硬盘的大小 ，-h表示可读的 
: 1708569006:0;du -d 1 -h    #命令查看当前目录下所有文件夹的大小 -d 指深度，后面加一个数值 # du -sh *
: 1708569007:0;git update-index --assume-unchanged file# 执行忽略指令
: 1708569008:0;git update-index --no-assume-unchanged file# 执行恢复指令

# 执行忽略指令: $ git update-index --assume-unchanged BinPodfile
# 执行恢复指令: $ git update-index --no-assume-unchanged BinPodfile
```

## /etc/ssh/sshd_config

```

HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
```

## zprofile 




```

  export HOMEBREW_PIP_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple #ckbrew
  export HOMEBREW_API_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api  #ckbrew
  export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles #ckbrew
  eval $(/opt/homebrew/bin/brew shellenv) #ckbrew


```

## podfile

```
#ruby2.7.0
# export PATH="/usr/local/opt/ruby/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/ruby/lib"
export CPPFLAGS="-I/usr/local/opt/ruby/include"
export PKG_CONFIG_PATH="/usr/local/opt/ruby/lib/pkgconfig"
# export PATH="/usr/bin/ruby/bin:$PATH"
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

## fastlane/FastFile


```
default_platform(:ios)
platform :ios do
  desc "Description of what the lane does"
  lane :create33 do
    produce(
      username: 'user@team.com',
      app_identifier: 'com.wlqq.driver.liveactivity',
      app_name: 'app',
      team_name: 'Guiyang Huochebang Technology Co.,Ltd', # only necessary when in multiple teams
      skip_itc: true, 
  
      # Optional
      # App services can be enabled during app creation
      enable_services: {
        game_center: "off", 
        push_notification: "on",                  # Valid values: #"on", "off"
        access_wifi: "on",                        # Valid values: #"on", "off"
        associated_domains: "on",                         # Valid #values: "on", "off"
        app_group: "on",  
      }
    )
  end
end
```

## .zshrc

```
zstyle ':omz:update' mode disabled 



alias opss="open . -a Sourcetree" #使用 Sourcetree 打开当前目录
alias opw="open *.xcworkspace" #iOS开发专用
#函数：
acps() {git aa;git cmm $1;gps}; #将当前全部本地修改提交、并push到远程分支、需要填入commit信息，例如 acps "feat：add feature"
adcm() {git aa;git cmm $1}; #同上，但是不会执行提交远程操作
sbl() {open $1 -a Sublime\ Text}; #使用 Sublime 打开指定文件，例如 sbl ~/.zshrc
mcd() {md $1;cd $1}; #创建一个文件夹并进入该文件夹
taga() {git fp; git tag $1; git pst}; #创建一个新tag，并同步至远程，创建之前会先移除远程已经删除但本地还存在的tag，避免重复推送tag
nbs() {git cob $1; git pso $1}; #创建一个新分支，并将它同步至远程
rbo() {git rbr $1; git rbro :$1}; #删除指定的本地和远程分支
# Add RVM to PATH for scripting. Make sure this is the last PATH variable change.
# export PATH="$PATH:$HOME/.rvm/bin"
# export PATH="/usr/local/opt/ruby/bin:$PATH"
alias expodupdate="cd Example;rm -rf Pods/; pod update;cd -" 
alias expodinstall="cd Example;rm -rf Pods/; pod install;cd -" 
alias exopen="cd Example;opw;cd -" 
```

## .git
.gitignore_global
```
*~
.DS_Store
contents.xcworkspacedata
IDEWorkspaceChecks.plist
*.xcbkptlist
*.xcuserstate

Podfile.orig

```

.gitignore
```
##### 根目录的忽略文件
.CFUserTextEncoding
.DS_Store
.Trash
.V2rayU
.cocoapods/
.cups/
.eclipse/
.fastlane/
.gem/
.git/
#.gitconfig
#.gitflow_export
#.gitignore
#.gitignore_global
#.hgignore_global
.npm/
#.npmrc
#.nrmrc
.oh-my-zsh/
#.profile
#.ssh
#.stCommitMsg
.swiftpm/
.viminfo
.vscode/
.zcompdump-MacBook Pro Shixuan-5.8
.zcompdump-MacBook Pro-5.8
.zcompdump-shixuan-5.8
.zcompdump-别施轩的MacBook Pro-5.8
#.zhistory
#.zprofile
#.zsh_history
.zsh_sessions/
#.zshrc
Desktop/
Documents/
Downloads/
Library/
Movies/
Music/
Pictures/
Public/
Sites/
alias
mb-specrepo/
mbrepos/
me/
swiftReport.html
test.py
test.ruby
test.sh
update_result.txt

```

.gitconfig
```
[alias]
	resetsoft = "!git reset --soft HEAD~"
    diffpodf = !git --no-pager diff --unified=0 Podfile
    diffpods = !git --no-pager diff --unified=0 *.podspec
    diffpodsv = !git diffpods | awk '$0~/\\+/' | awk '$0~/version/' | awk -F \"=\" '{print $2}'
    pullrebase = "!git add *; git stash; git pull --rebase; git stash pop; git diffpodf; git diffpods"
    commitpodf = "!git commit Podfile -m \"Podfile Update`git diffpodf | awk '$0~/\\+.*pod/' | awk -F \"pod\" '{print $2}'`\""
    commitpods = "!git commit *.podspec -m \"Podspec Update `git diffpods | awk '$0~/\\+/' | awk '$0~/version/' | awk -F \"=\" '{print $2}'`\""
    commitpodsall = "!messagepods=\"Podspec Update `git diffpodsv`\";git add *.podspec; git commit -m \"$messagepods\""
    tagandupdate = "!git tag "
#    diffpodf2 = !git --no-pager diff --unified=0 Example/Podfile
#	commitpodf2 = "!git commit Example/Podfile -m \"Podfile Update `git diffpodf2 | awk '$0~/\\+  pod/' | awk -F \"pod\" '{print $2}' | awk -F \",\" '{print $1}'`\""
# | awk -F \",\" '{print $1}'
# | awk -F \",\" '{print $1}'
    rbr = branch -D #删除指定本地分支
    rbro = push origin #删除远程分支 例如运行 git rbro :feat/test 来删除远程的 feat/test 分支
    cob = checkout -b #新建分支
    co = checkout #切换分支
    aa = add . #添加当前所有修改文件
    mr = merge 
    cm = commit
    cmm = commit -m
    reh = reset --hard #强制重置到指定位置，不带参数即可重置当前本地所有修改
    res = reset --soft #重置到指定位置并保留所有修改
    fp = fetch -P -p
    po = pull origin #拉取远程分支
    ps = push 
    pst = push --tag
    psf = push -f #强制提交
    pso = push --set-upstream origin #将本地分支提交至远程
    md = commit --amend #修改上次提交
    mdn = commit --amend --no-edit #修改上次提交并直接使用上次提交的commit信息
	 
[core]
	excludesfile = /Users/admin/.gitignore_global
[difftool "sourcetree"]
	cmd = opendiff \"$LOCAL\" \"$REMOTE\"
	path = 
[mergetool "sourcetree"]
	cmd = /Applications/Sourcetree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
	trustExitCode = true
[user]
	name = user
	email = user@team.com
[commit]
	template = /Users/user/.stCommitMsg
# [branch]
# 	rebase = true
# [diff]
# 	context = 5
[pull]
[init]
	defaultBranch = master
[safe]
	directory = /opt/homebrew
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-core/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-cask/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-services/
	directory = /opt/homebrew
	directory = /opt/homebrew
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-core/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-cask/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-services/
	directory = /opt/homebrew
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-core/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-cask/
	directory = /opt/homebrew/Library/Taps/homebrew/homebrew-services/
[filter "lfs"]
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true

```

## 关于git命令行部分

### .gitconfig

默认配置一些全局的 git alias，var 等

```
[alias]
    resetsoft = "!git reset --soft HEAD~"
    diffpodf = !git --no-pager diff --unified=0 Podfile
    diffpods = !git --no-pager diff --unified=0 *.podspec
    pullrebase = "!git add *; git stash; git pull --rebase; git stash pop; git diffpodf; git diffpods"
    commitpodf = "!git commit Podfile -m \"Podfile Update`git diffpodf | awk '$0~/\\+  pod/' | awk -F \"pod\" '{print $2}'`\""
    commitpods = "!git commit *.podspec -m \"Podspec Update vsersion`git diffpods | awk '$0~/\\+/' | awk '$0~/version/' | awk -F \"=\" '{print $2}'`\""
#    diffpodf2 = !git --no-pager diff --unified=0 Example/Podfile
#    commitpodf2 = "!git commit Example/Podfile -m \"Podfile Update `git diffpodf2 | awk '$0~/\\+  pod/' | awk -F \"pod\" '{print $2}' | awk -F \",\" '{print $1}'`\""
# | awk -F \",\" '{print $1}'
# | awk -F \",\" '{print $1}'
    rbr = branch -D #删除指定本地分支
    rbro = push origin #删除远程分支 例如运行 git rbro :feat/test 来删除远程的 feat/test 分支
    cob = checkout -b #新建分支
    co = checkout #切换分支
    aa = add . #添加当前所有修改文件
    mr = merge 
    cm = commit
    cmm = commit -m
    reh = reset --hard #强制重置到指定位置，不带参数即可重置当前本地所有修改
    res = reset --soft #重置到指定位置并保留所有修改
    fp = fetch -P -p
    po = pull origin #拉取远程分支
    ps = push 
    pst = push --tag
    psf = push -f #强制提交
    pso = push --set-upstream origin #将本地分支提交至远程
    md = commit --amend #修改上次提交
    mdn = commit --amend --no-edit #修改上次提交并直接使用上次提交的commit信息
     
[core]
    excludesfile = /Users/user/.gitignore_global
    fileMode = false
[difftool "sourcetree"]
    cmd = opendiff \"$LOCAL\" \"$REMOTE\"
    path = 
[mergetool "sourcetree"]
    cmd = /Applications/Sourcetree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
    trustExitCode = true
[user]
    name = user
    email = user@team.com
[commit]
    template = /Users/user/.stCommitMsg
# [branch]
#     rebase = true
# [diff]
#     context = 5

```

### 获取 tag， commitid，message， branch 等展示信息

#### --no-pager
一个非常有用的参数，可以将 pager 的输出转换为 --no-pager 输出。

```
git --no-pager tag
```

#### 展示所有 tag 信息

```
git show tag1 
```

##### 通过 commitid 找 tag
可能多个
```
tag_names=$(git describe --tags $last_tag_commit)
```

#### 通过 tag 找 tag message / commit message
没有 tag message 的情况下会输出 commit message
```
git --no-pager tag -l --format='%(contents)' tag2
```

#### 通过 tag 找 commitid
rev-list 可以输出 commitid 信息，取第一个即可
```
git rev-list tag1  --max-count=1
```
或者直接使用参数 rev-parse
```
git rev-parse tag2
```
rev-parse 还可以使用参数 --short HEAD，获取默认7位的短 id

```
git rev-parse tag2 --short HEAD
```

#### 全局最新一个 commitid
同上，会打印日期最新的commit，而不是当前分支的

```
git rev-list --all --max-count=1
```

#### 全局最新一个 tag
同上，然后调用 commit id 取 tag 即可。

#### 展示所有 分支 信息
打印所有分支
```
git --no-pager branch
```

#### 当前分支
输出当前分支名

```
git symbolic-ref --short -q HEAD
```

或者 
```
git rev-parse --abbrev-ref HEAD
```

#### 远程标签
对于标签来说，每一个远程标签会有个特色的 refid，而不是只是有 commitid

```
git show-ref --tags
```
看到 --tags，可以联想到也能够使一些其他的参数命令了。

#### 查看远程标签和本地标签
本地标签 和 所有标签。
```
cat .git/refs/tags/* 

cat .git/packed-refs
```

#### log --format="%ct"
log命令，支持输出时间戳，所以上面的输出如果有 commitid，即可使用 log --format="%ct" 输出时间戳。
```
git log -1 --format="%ct"
```

#### 通过 log 查看两个节点之间的 commit 信息

```
git log tag1 tag2
git log commitid1 commitid2
git log --pretty=oneline ^tag1 tag2
```

#### 通过 diff 查看两个节点之间的修改信息

```
git diff tag1 tag2
git diff commitid1 commitid2
git --no-pager diff --unified=0 commitid1 commitid2
```
只显示修改的文件
```
git diff --diff-filter=M
```

### 修改提交 commit tag


#### 自动生成提交 message
如上面 config 部分，设置使用即可。

#### 提交tag
简单的提交tag的信息。

```
git tag "stable/"$dev_branch-$commit_id $commit_id -m "stable tag"
git push origin "stable/"$dev_branch-$commit_id
```

#### 批量删除 tag
坑点注意，本地 tag 的信息如果被删除，那没救无法获取远程 ref tag 信息了。

```
###- 刷新仓库
git fetch a_origin --prune

###- 检查最后一个tag是否是stable tag
last_tag_commit=$(git rev-list --tags --max-count=1)
last_tag_name=$(git describe --tags $last_tag_commit)
echo $last_tag_name

if [[ $last_tag_name =~ "stable" ]]; then
    ###- 清理远程 stable tag
    for line in $(git show-ref --tag | awk '/stable/ {print ":" $2}'); do
        # echo $line
        if [[ $line =~ $last_tag_name ]]; then
            echo "ignore current tag $line"
        else
            echo "git push a_origin $line"
            git push a_origin $line
        fi
    done
    ##- 清理本地 stable tag
    git tag -l | awk '/stable/' | xargs git tag -d
fi
```

#### reset
如开头的 config 部分
```
git reset --hard #强制重置到指定位置，不带参数即可重置当前本地所有修改
git reset --soft #重置到指定位置并保留所有修改
```

#### --no-pager
如开头的 config 部分
```
git commit --amend #修改上次提交
git commit --amend --no-edit #修改上次提交并直接使用上次提交的commit信息
```

#### 提交 mr 信息

```
git push -v -o merge_request.create -o merge_request.target=$branch1 2>&1
```

#### merge

```
git merge rel/20240229
git merge origin/rel/20240229 -no-ff
```
#### reset rebase

通过 reset rebase 可以解决一些 merge 代码无法回退的问题：

1、reset hard 到 merged commit
2、branch -b 新建备份分支
3、reset soft 到上个合并前节点
4、提交 commit
5、merge 到合并后分支
6、revert commit
7、merge revert的commit


```
git reset origin/develop
git rebase origin/develop

```