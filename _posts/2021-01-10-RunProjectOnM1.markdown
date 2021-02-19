---
layout:     post                       # 使用的布局（不需要改）
title:      "在 M1 电脑上运行，调试项目和打包ipa" # #重回 Layout                  # 标题
subtitle:   项目运行, xcframework, PackageApplication, 打包 ipa 给 M1           #副标题
date:       2021-01-10                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 总结
- 测试
---


### 简介

本文主要是两个方面的内容：

- **项目迁移**
- **打包 ipa 给 M1**

在 Xcode 12 安装完成之后，你就会发现，在 M1 电脑上运行并不难...

如果你的项目已经迁移到 Xcode 12 的话...

根据实践，问题大概像下边：

- 主工程迁移到 Xcode 12
- 三方依赖库迁移到 Xcode 12
- 自己依赖库迁移到 Xcode 12
- 依赖库迁移到 xcframework

所以会发现，大量时间是在做项目迁移。然后是：

- **项目在 mac 电脑运行调试**
- **打包测试版 ipa**

[Apple Developer Documents](https://developer.apple.com/documentation/apple_silicon/running_your_ios_apps_on_macos)

[Other Documents](https://www.theverge.com/2020/11/18/21574207/how-to-install-run-any-iphone-ipad-app-m1-mac)

### 项目迁移到 Xcode 12

迁移方面，除了一些配置的修改，更多的是跟 framework 相关。如果使用了 framwork，那么就会出现打包的 frame 架构的交叉重叠的问题。

#### 关于架构

```
arm6: iPhone 1, 2, 3G
arm7: Used in the oldest iOS 7-supporting devices[32 bit]. iPhone 4, 4s
arm7s: As used in iPhone 5 and 5C[32 bit]. iPhone 5, 5c
arm64: For the 64-bit ARM processor in iPhone 5S[64 bit]. iPhone 5s and above.
arm64: On the arm mac, the Xcode12 simulator also uses arm64, but it is not shared with the arm64 of the real device.
arm64e: used on the A12 chipset. iPhone XS/XS Max/XR
i386: For the 32-bit simulator
x86_64: Used in 64-bit simulator
```
关于 arm64, 在 M1 的电脑上，模拟器也变成了 arm64 架构了，所以 xcode 12 的 framework 在打包模拟器的时候会默认多build一个 arm64在里面。但是这跟 iOS 真机的 arm64 并不通用。

这就导致了在合并 framework 的时候就会很尴尬，如果你选择去掉模拟器的 arm64，那么在真机运行不会有问题，但是在模拟器上运行就会报错。

这个问题的最终版本解决方式就是使用 xcframework。

#### xcframework

xcframework 是 WWDC 2019 推出的新的 framework 格式。它的出现，几乎完全就是为了解决架构问题。

跟 framework 相比，突出的优点：
- 把所有平台的 framework 包都包含了
- 项目的依赖☑️，只需要连接 xcframework 即可，不用区分
- 不必 thin 和 universal
- 不必在上传 appstore 之前去掉 x86_64

文件夹的结构如下，本质就是用了个集合，将所有使用的架构都放到文件夹里面，根据打包再一一分开：

```
Info.plist
ios-arm64
ios-arm64_x86_64-simula
```

话不多说，提供一个打包介绍：


1. 首先到要打包的 project， 修改配置

BUILD_LIBRARY_FOR_DISTRIBUTION YES

2. 使用脚本打包：

```
cd FrameworkProject                               
rm -rf build
mkdir build

xcodebuild archive -scheme FrameworkProject -destination="iOS" -archivePath build/iphoneos -derivedDataPath build/iphoneos -sdk iphoneos SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

xcodebuild archive -scheme FrameworkProject -destination="iOS Simulator" -archivePath build/iphonesimulator -derivedDataPath build/iphonesimulator  -sdk iphonesimulator SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

rm -rf build/FrameworkProject.xcframework

xcodebuild -create-xcframework -framework build/iphoneos.xcarchive/Products/Library/Frameworks/Lottie.framework -framework build/iphonesimulator.xcarchive/Products/Library/Frameworks/FrameworkProject.framework -output build/FrameworkProject.xcframework
```


关于 xcframework：
https://medium.com/trueengineering/xcode-and-xcframeworks-new-format-of-packing-frameworks-ca15db2381d3


### 项目在M1的运行和调试

项目勾选`My Mac (Designed for iPad)`即可在M1 mac上运行 app程序。

#### 调试

调试基本上跟真机调试很像，有一些常见的地方，列出来了：

根据测试，大部分功能可以正常使用：

- touch id 可以使用电脑的 touch id， 摄像头会使用前置摄像头。 *在 M1 macbook pro测试*
- 多数自动布局的 UI 是没有问题的
- 使用自动填充的 password 之类的也可以使用 mac 电脑的 keychain 之类
- 在外部打开的 webview 会在电脑 safari 打开，并且可以使用 universal link 回到 app

当然也会有一些问题：

- 一些 UI 不规范导致的问题，字体被遮盖等
- 输入框的 toolbar 会始终显示在屏幕最下边，因为不会有键盘
- 如果有摄像头，取景区域是否正确。
- 推送的问题等

#### 打包测试包给 M1 mac 测试

网上很少有这方面的资料，大部分给出的意见都是通过 app store 下载 release 的包。

那么如何**打包测试包给电脑使用**呢：

尝试使用了 archive 的 iphone 包，可以在真机上安装但是不能在 M1 电脑安装。安装包已经损坏。

最终使用了 build 的 debug 包，通过早已经失传的 PackageApplication 导出 build 的 ipa 包，经过测试发现可以安装在 M1 mac。


配置：

1. 创建 PackageApplication

```
vi PackageApplication
```

2. 复制到 Xcode.app, 选择 command line

```
cp PackageApplication /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin

sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/

chmod +x /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/PackageApplication
```

使用：

1. build 真机包

2. 导出真机包到 ipa

```
xcrun -sdk iphoneos PackageApplication -v /Users/user/Library/Developer/Xcode/DerivedData/Project-gxnibnoowpeswqcgrurzvrgpiels/Build/Products/Debug-iphoneos/RushCard.app -o /Users/user/Desktop/Project.ipa\
```

附上 PackageApplication：



```
#!/usr/bin/perl
#
#    PackageApplication
#   
#    Copyright (c) 2009-2012 Apple Inc.  All rights reserved.
#
#    Package an iPhone Application into an .ipa wrapper
#
use Pod::Usage;
use Getopt::Long;
use File::Temp qw/ tempdir :POSIX /;
use File::Basename;
use File::Path qw/ remove_tree /;
use Cwd qw/ chdir /;
print "\n\n\nwarning: PackageApplication is deprecated, use `xcodebuild -exportArchive` instead.\n\n\n";
$|=1;   # don't buffer stdout
my $program = $0;
my %opt = ();
GetOptions ( \%opt,
             "sign|s=s",
             "embed|e=s",
             "output|o=s",
             "symbols=s",
             "verbose|v!",
             "help|h|?",
             "man",
             "plugin=s@" => \@plugins,
             ) or pod2usage(2);
pod2usage(2) if $opt{help};
pod2usage(-exitstatus => 0, -verbose => 2) if $opt{man};
fatal("An application was not specified.") unless $ARGV[0];
my $origApp = shift @ARGV;
fatal("Specified application doesn't exist or isn't a bundle directory : '$origApp'") unless -d $origApp;
if ( $opt{verbose} ) {
    print "Packaging application: '$origApp'\n";
    print "Arguments: ";
    while( ($key,$value) = each %opt)
    {
        print "$key=$value  ";
    }
    print "\n";
    print "Environment variables:\n";
    while( ($key,$value) = each %ENV)
    {
        print "$key = $value\n";
    }
    print "\n";
}
# check any plugins that might be specified
foreach $plugin (@plugins) {
    print "Plugin: '$plugin'\n" if $opt{verbose};
    fatal("Specified plugin doesn't exist or isn't a bundle directory : '$plugin'") unless -d $plugin;
}
# setup the output name if it isn't specified
# setup the output if it isn't specified
if ( !defined($opt{output})  ) {
    $opt{output} = dirname($origApp).'/'.basename($origApp, ".app")."\.ipa";
}
print "Output directory: '$opt{output}'\n" if $opt{verbose};
# Make sure we have a temp dir to work with
my $tmpDir = tempdir( CLEANUP => !defined($opt{verbose}) );
print "Temporary Directory: '$tmpDir'  (will NOT be deleted on exit when verbose set)\n" if $opt{verbose};
################## Start Packaging #####################
### Step One : Make a copy of the application (and any plugins)
my $appName = basename($origApp);
chomp $appName;
my $destAppDir = "$tmpDir/Payload";
my $destApp = "$destAppDir/$appName";
mkdir $destAppDir;
fatal("Unable to create directory : '$destAppDir'") unless -d $destAppDir;
runCmd("/bin/cp", "-Rp", $origApp, $destAppDir);
fatal("Unable to copy application '$origApp' into '$destAppDir'") unless -e $destApp;
foreach $plugin (@plugins) {
    my $pluginName = basename($plugin);
    chomp $pluginName;
    my $destPlugin = "$destAppDir/$pluginName";
    my $result = runCmd("/usr/bin/codesign", "--verify", "-vvvv", , $plugin );
    if ( $result !~ /valid on disk/ ) {
        fatal("Codesign check fails : $result\n");
    }
    runCmd("/bin/cp", "-Rp", $plugin, $destAppDir);
    fatal("Unable to copy application '$plugin' into '$destAppDir'") unless -e $destPlugin;
}
if ( $opt{symbols} ) {
    my $destSymbols = "$tmpDir/Symbols";
    runCmd("/bin/cp", "-Rp", $opt{symbols}, $destSymbols);
    fatal("Unable to copy symbols '$opt{symbols}' into '$destSymbols'") unless -e $destSymbols;
}
### Step Two : recode sign it if necessary
if ( $opt{verbose} ) {
    print "### Checking original app\n";
    my $result = runCmd("/usr/bin/codesign", "--verify", "-vvvv", , $origApp );
    if ( $result !~ /valid on disk/ ) {
        print "Codesign check fails : $result\n";
    }
    print "Done checking the original app\n";
}
if ( defined $opt{sign} ) {
    if ( $opt{embed} ) {
        print "### Embedding '$opt{embed}'\n" if $opt{verbose};
        runCmd("/bin/rm", "-rf", "$destApp/embedded.mobileprovision" );
        fatal("Unable to remove '$destApp/embedded.mobileprovision'\n") if ( -e "$destApp/embedded.mobileprovision" );
        runCmd("/bin/cp", "-rp", $opt{embed}, "$destApp/embedded.mobileprovision");
        fatal("Unable to copy '$opt{embed}' to '$destApp/embedded.mobileprovision'\n") unless ( -e "$destApp/embedded.mobileprovision" );
    }
    my $entitlements_plist = File::Temp::tempnam($tmpDir, "entitlements_plist");
    # If re-signing with a distribution profile and get-task-allow is
    # true, the app store will reject the submission. Setting
    # get-task-allow to false here.
    if ( $opt{sign} =~ /^i(Phone|OS) Distribution/ ) {
        my $entitlements_raw = File::Temp::tempnam($tmpDir, "entitlements_raw");
        runCmd("/usr/bin/codesign", "-d", "--entitlements", $entitlements_raw, $destApp);
        $? == 0 or fatal("Failed to read entitlements from '$destApp'");
        if ( -e $entitlements_raw ) {
            $plist = read_raw_entitlements($entitlements_raw);
            unlink($entitlements_raw) or fatal("Cannot delete '$entitlements_raw': $!");
            open(my $ofh, '>', $entitlements_plist) or fatal("Cannot open '$entitlements_plist': $!");
            print $ofh $plist or fatal("Cannot write entitlements to '$entitlements_plist': $!");
            close($ofh) or fatal("Cannot close file handle for '$entitlements_plist': $!");
            runCmd('/usr/libexec/PlistBuddy', '-c', 'Set :get-task-allow NO', $entitlements_plist);
            # PlistBuddy will fail if get-task-allow doesn't exist. That's okay.
            runCmd('/usr/bin/plutil', '-lint', $entitlements_plist);
            $? == 0 or fatal("Invalid plist at '$entitlements_plist'");
        }
    }
    my @codesign_args = ("/usr/bin/codesign", "--force", "--preserve-metadata=identifier,entitlements,resource-rules",
                         "--sign", $opt{sign},
                         "--resource-rules=$destApp/ResourceRules.plist");
    if ( -e $entitlements_plist ) {
        push(@codesign_args, '--entitlements');
        push(@codesign_args, $entitlements_plist);
    }
    push(@codesign_args, $destApp);
    print "### Codesigning '$opt{embed}' with '$opt{sign}'\n" if $opt{verbose};
    my $codesign_output = runCmd(@codesign_args);
    $? == 0 or fatal("@codesign_args failed with error " . ($? >> 8) . ". Output: $codesign_output");
    if ( -e $entitlements_plist ) {
        unlink($entitlements_plist) or fatal("Cannot delete '$entitlements_plist': $!");
    }
}
### Step Three : zip up the package
remove_tree("$opt{output}");
fatal("Unable to remove older '$opt{output}'") if ( -e "$opt{output}" );
chdir $tmpDir;
if($opt{verbose}) {
    runCmd("/usr/bin/zip", "--symlinks", "--verbose", "--recurse-paths", "$opt{output}", ".");
} else {
    runCmd("/usr/bin/zip", "--symlinks", "--quiet", "--recurse-paths", "$opt{output}", ".");
}
fatal("Unable to create '$opt{output}'") if ( ! -e "$opt{output}" );
print "Results at '$opt{output}' \n" if $opt{verbose};
################## Finished ############################
exit 0;
########################################################
sub fatal {
    my ($msg) = @_;
    print STDERR "error: $msg\n";
    exit 1;
}
use POSIX qw(:sys_wait_h);
sub runCmd {
    my (@cmds) = @_;
    my $output = ();
    if ( $opt{verbose} ) {
        my $_cmd = join(" ", @cmds);
        print "+ $_cmd\n" ;
    }
    my $program = shift @cmds;
    my ($readme, $writeme);
    pipe $readme, $writeme;
    my $pid = fork;
    defined $pid or die "cannot fork: $!";
    if ( $pid == 0 ) {
        # child
        open(STDOUT, ">&=", $writeme) or die "can't redirect STDOUT: $!";
        open(STDERR, ">&=", $writeme) or die "can't redirect STDERR: $!";
        close $readme;
        exec($program, @cmds) or die "can't run $program: $!";
    }
    close $writeme;
    while(<$readme>) {
        $output .= $_;
    }
    close $readme;
    my $waitpid_ret;
    do { $waitpid_ret = waitpid($pid, 0); } while ( $waitpid_ret == 0 );
    $waitpid_ret == $pid or die "waitpid returned $waitpid_ret: $!";
    my $return_code = $? >> 8;
    $opt{verbose} and print "Program $program returned $return_code : [$output]\n";
# Let clients handle error checking.
#    $return_code == 0 or fatal("Program $program failed with return code $return_code.");
    return $output;
}
# Param: path to codesign --entitlements output file. Returns xml plist in a string.
sub read_raw_entitlements {
    my $entitlements_raw = shift;
    open(my $ifh, '<', $entitlements_raw) or fatal("Cannot open '$entitlements_raw': $!");
    read($ifh, my $format_tag, 4) or fatal("Cannot read format tag from '$entitlements_raw': $!");
    $format_tag = unpack('H*', $format_tag);
    my $expected_format_tag = 'fade7171';
    $format_tag eq $expected_format_tag
        or fatal("Format tag '0x$format_tag' does not match '0x$expected_format_tag'.");
    read($ifh, my $plist_size, 4) or fatal("Cannot read plist size from '$entitlements_raw': $!");
    $plist_size = unpack('L>', $plist_size);
    read($ifh, my $plist, $plist_size) or fatal("Cannot read $plist_size bytes of plist data from '$entitlements_raw': $!");
    close($ifh) or fatal("Cannot close file handle for '$entitlements_raw': $!");
    return $plist;
}
__END__
=head1 NAME
PackageApplication - prepare an application for submission to AppStore or installation by iTunes.
=head1 SYNOPSIS
PackageApplication [-s signature] application [-o output_directory] [-verbose] [-plugin plugin] || -man || -help
Options:
    -s <signature>  certificate name to resign application before packaging
    -o              specify output filename
    -plugin         specify an optional plugin
    -help           brief help message
    -man            full documentation
    -v[erbose]      provide details during operation
=head1 OPTIONS
=over 8
=item B<-s>
Optional codesigning certificate identity common name.  If provided, the application is will be re-codesigned prior to packaging.
=item B<-o>
Optional output filename.  The packaged application will be written to this location.
=item B<-plugin>
Specify optional plugin.  The packaged application will include the specified plugin(s).
=item B<-help>
Print a brief help message and exits.
=item B<-man>
Prints the manual page and exits.
=item B<-verbose>
Provides additional details during the packaging.
=back
=head1 DESCRIPTION
This program will package the specified application for submission to the AppStore or installation by iTunes.
=cut
```


### 总结

本文主要介绍如何升级项目，如何打包 m1 ipa，大家觉得好的话，不妨点个赞。哈哈哈，谢啦。
