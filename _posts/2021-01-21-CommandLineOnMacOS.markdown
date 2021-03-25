---
layout:     post
title:      归纳总结一下最近常用的命令行
subtitle:   Jekyll, Git, Convert mov to mp4, imagemagick, compression, HostName
date:       2021-01-21
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- Script
---

### 简介

纯干货篇～


#### Jekyll

jekyll serve -w --host=0.0.0.0


#### Git ignore FileMode

git config core.filemode false

or

git config --global core.fileMode false  


#### convert mov to mp4

`brew install ffmpeg`

```
ffmpeg -i input.mov -acodec copy -vcodec copy output.mp4
```


#### imagemagick convert image to long image

`brew install imagemagick`

```
convert `find . -name '*.png' | sort` +append r.png
```


#### Dev - lipo universal framework
*Deprecated*

```
lipo -create -output yourframework.framework yourframeworkDevice.framework/yourframeworkDevice yourframeworkSim.framework/yourframeworkSim
```


#### Git auto save on TC

```
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
```


#### image compression

`brew install pngquant`

```
pngquant input.png
```


#### image compression and convert

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


#### Dev - Xcode achieve and export

`brew install pngquant`

```
# archive and export
#!/bin/bash

# Archive
xcodebuild -scheme GoBank -archivePath ./MyApp.xcarchive archive -sdk iphoneos

# Export Archive
xcrun xcodebuild -exportArchive -exportOptionsPlist ./exportPlist.plist -archivePath ./MyApp.xcarchive -exportPath ./MyApp.ipa

```

exportPlist.plist:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>teamID</key>
	<string>XXXXXXXXX</string>
	<key>method</key>
	<string>development</string>
	<key>uploadSymbols</key>
	<true/>
	<key>provisioningProfiles</key>
	<dict>
		<key>com.example.example</key>
		<string>APP Dev</string>
	</dict>
</dict>
</plist>

```


#### Dev - Swift Package

`swift package init --type executable`

```
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
```

`swift build -c release`


#### Dev - Build XCFramework

```
cd xxx                               
rm -rf build
mkdir build


xcodebuild archive -scheme Framework_Project -destination="iOS" -archivePath build/iphoneos -derivedDataPath build/iphoneos -sdk iphoneos SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES


xcodebuild archive -scheme Framework_Project -destination="iOS Simulator" -archivePath build/iphonesimulator -derivedDataPath build/iphonesimulator  -sdk iphonesimulator SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES


rm -rf build/Framework_Project.xcframework


xcodebuild -create-xcframework -framework build/iphoneos.xcarchive/Products/Library/Frameworks/Lottie.framework -framework build/iphonesimulator.xcarchive/Products/Library/Frameworks/Lottie.framework -output build/Framework_Project.xcframework

```


#### Dev - Set PC HostName

Get your HostName with:
```
scutil --get HostName
HostName: not set
```

Set your HostName with:
```
sudo scutil --set HostName 'yourHostName' 
```

### 结束

完～
