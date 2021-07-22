---
layout:     post
title:      Note 一些杂七杂八的脚本命令行
hide-in-nav: true
subtitle:   
date:       2021-06-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Notes
---


<!-- test  -->

#shutdown
xcrun simctl shutdown 'iPhone 11 Pro Max'
#reset
xcrun simctl erase 'iPhone 11 Pro Max' 

############################################

jekyll serve -w --host=0.0.0.0

############################################

git config core.filemode false

############################################

git config --global core.fileMode false  

############################################

ffmpeg -i input.mov -acodec copy -vcodec copy output.mp4

############################################

cd /Users/shixuan.bie/Perforce/RushCard-Legacy-Dev_7674

rm -rf Source/Libraries/UJET/Slimmed/*

############################################

#convert
convert a1.png a2.png a3.png +append r.png

############################################

lipo -create -output yourframework.framework yourframeworkDevice.framework/yourframeworkDevice yourframeworkSim.framework/yourframeworkSim

############################################
#!/bin/bash

if [ -z $1 ]
then 
	echo "Usage: sh mergeparent.sh {REPO_NAME}"
	exit 1
fi

git fetch 
git rebase origin/master
git fetch $1 master
git merge fetch_head
git mergetool
git clean -f
git commit -a -m "merge $1"
git push origin master
############################################

# image compression and convert
#!/bin/bash

echo "re-mkdir new/"
rm -rf new/
mkdir new/

echo "pngquant imgs"
find . -name '*.png' | xargs pngquant --quality=10-50

echo "imgs move to new/"
find . -name '*-fs8.png' -print0 | xargs -0 -I {}  mv {} new/

echo "convert new/"
convert `find new/ -name '*.png' | sort` +append new/r.png
############################################

# archive and export
#!/bin/bash

# Archive
xcodebuild -scheme GoBank -archivePath ./MyApp.xcarchive archive -sdk iphoneos

# Export Archive
xcrun xcodebuild -exportArchive -exportOptionsPlist ./exportPlist.plist -archivePath ./MyApp.xcarchive -exportPath ./MyApp.ipa

￼
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>teamID</key>
	<string>teamID</string>
	<key>method</key>
	<string>app-store</string>
	<key>uploadSymbols</key>
	<true/>
	<key>provisioningProfiles</key>
	<dict>
		<key>com.example.app</key>
		<string>App Name</string>
	</dict>
</dict>
</plist>
############################################

swift package init --type executable


// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "ThemeScript",
    // products: [
    //     // Products define the executables and libraries a package produces, and make them visible to other packages.
    //     .library(
    //         name: "ThemeScript",
    //         targets: ["ThemeScript"]),
    // ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "ThemeScript",
            dependencies: ["ThemeScriptCore"],
            path: "ThemeScript"),
        .target(
            name: "ThemeScriptCore",
            path: "ThemeScriptCore"),
        .testTarget(
            name: "ThemeScriptTests",
            dependencies: ["ThemeScriptCore"]),
    ]
)




swift build -c release
############################################


lipo -info xxx.framework/xxx


Build XCFramework

BUILD_LIBRARY_FOR_DISTRIBUTION YES




cd ~/Folder                   

cd lottie                         
rm -rf build
mkdir build


xcodebuild  -target Lottie -configuration Release -sdk iphoneos ENABLE_BITCODE=YES BITCODE_GENERATION_MODE=bitcode ONLY_ACTIVE_ARCH=NO BUILD_DIR=build BUILD_ROOT=build clean build



xcodebuild  -target Lottie -configuration Debug -sdk iphonesimulator ENABLE_BITCODE=YES BITCODE_GENERATION_MODE=bitcode ONLY_ACTIVE_ARCH=NO BUILD_DIR=build BUILD_ROOT=build clean build



rm -rf build/Lottie.xcframework

xcodebuild -create-xcframework \
-framework build/Debug-iphonesimulator/Lottie.framework \
-framework build/Release-iphoneos/Lottie.framework \
-output build/Lottie.xcframework

// —————————————————————————

cd ~/Floder                 

cd lottie                        
rm -rf build
mkdir build


xcodebuild archive -scheme Lottie_iOS -destination="iOS" -archivePath build/iphoneos -derivedDataPath build/iphoneos -sdk iphoneos SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES


xcodebuild archive -scheme Lottie_iOS -destination="iOS Simulator" -archivePath build/iphonesimulator -derivedDataPath build/iphonesimulator  -sdk iphonesimulator SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES


rm -rf build/Lottie.xcframework


xcodebuild -create-xcframework -framework build/iphoneos.xcarchive/Products/Library/Frameworks/Lottie.framework -framework build/iphonesimulator.xcarchive/Products/Library/Frameworks/Lottie.framework -output build/Lottie.xcframework

############################################

Get your HostName with:
scutil --get HostName
HostName: not set
Set your HostName with:
sudo scutil --set HostName 'yourHostName' 
############################################

ios simulator directory

/Applications/Xcode_11.7.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer
/Applications/Xcode_12.4.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer

############################################

swift interface language

swiftc -emit-silgen -O main.swift  

############################################


#查看github
@code{.bash} cd ~/my_working_directory python opencv/platforms/ios/build_framework.py ios --contrib opencv_contrib --without optflow @endcode

#build open cv

python opencv/platforms/ios/build_framework.py ios


Using IPHONEOS_DEPLOYMENT_TARGET=9.0
Using iPhoneOS ARCHS=['armv7', 'armv7s', 'arm64']
Using iPhoneSimulator ARCHS=['i386', 'x86_64']

############################################

############################################

############################################

############################################

############################################