---
title: 填坑路之:ITMS-90171
categories: iOS
tages: [iOS,Xcode,ITMS-90171]
---
最近Swift和Objective-c混编使用脚本打包提交App Store遇到报错 ITMS-90171:"Invalid Bundle Structure - The binary file "xxx.app/libswiftRemoteMirror.dylib" is not permitted" 
# 不啰嗦 这里把解决问题放第一位，所以这里先贴打包脚本源码

```bash
TARGET="xxx" #这里一般填你的工程名字
PROJECT_NAME=$TARGET
PROJECT_FILE=$PROJECT_NAME.xcodeproj
CONFIGURATION=Release
BUILD_FOLDER=build/$CONFIGURATION-iphoneos
RESULT_FOLDER=result

echo "clear folder"
rm -rf $BUILD_FOLDER
mkdir -p $BUILD_FOLDER
rm -rf $RESULT_FOLDER
mkdir -p $RESULT_FOLDER

echo clean...
xcodebuild  -project  $PROJECT_FILE  -scheme  $PROJECT_NAME  -configuration  $CONFIGURATION clean

echo begin xcodebuild...
xcodebuild -project $PROJECT_FILE -scheme $PROJECT_NAME -configuration $CONFIGURATION  -archivePath $BUILD_FOLDER/$PROJECT_NAME.xcarchive  archive

echo begin xcrun...
xcrun xcodebuild -exportArchive -exportOptionsPlist build_config/archive_option.plist -archivePath  $BUILD_FOLDER/$PROJECT_NAME.xcarchive -exportPath $RESULT_FOLDER

echo $RESULT_FOLDER/$PROJECT_NAME.ipa
```
这里附上archive_option.plist文件内容
```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>teamID</key>
    <string>xxx</string>
    <key>uploadSymbols</key>
    <false/>
</dict>
</plist>
```
[xcrun命令](http://www.jianshu.com/p/bccbb37f21f9)参考

# 有闲情逸致的可以看看下面的解决过程
首先google之查到一些信息，libswiftRemoteMirror.dylib这货是Xcode挖的坑，Swift跟oc混编用Xcode8的.app打包就会自动给你放个libswiftRemoteMirror.dylib这货在包里面，然后你提交的时候它有告诉你不允许包里面有这货，这时候我就想骂人了，但是想了下英语除了会fuck就不会其它高雅一点的了，于是放弃骂人，还是继续埋头想办法吧...于是习惯性的在网上找到[一篇文章](http://www.jianshu.com/p/03e4a3597af5),这文章里面主要说了三个方法，先列出来
```bash
1.通过Xcode的Archive方式打包。Archive打的包，就没有包含这个libswiftRemoteMirror.dylib文件。
2.手动删除.app中的libswiftRemoteMirror.dylib文件，然后进行.app打包。
3.在Xcode7中进行.app打包。在Xcode7和Xcode8中分别进行.app打包，发现Xcode7打包后没有libswiftRemoteMirror.dylib文件
```
由于本人比较懒，能用脚本解决的事绝对不用鼠标点第二下，所以所有方法都是用脚本实现的
首先，方法3直接排除，尼玛苹果Xcode向下兼容做得最扯蛋的，Xcode8开发的项目还拿Xcode7去运行简直嫌自己被坑得不够惨吧！
然后看方法2感觉很简单的样子，于是在压缩成ipa的脚本前面加了这两行代码（删文件+重签）
```bash
rm -rf $BUILD_DIR/$PROJECT.app/libswiftRemoteMirror.dylib
codesign -f -s 'xxx xxx xx, Ltd (xxxx)' $BUILD_DIR/$PROJECT.app
```
搞定提交，以为完事可以回家打豆豆了，但是苹果君还想让我陪它玩玩，于是又跳出来个错误 ITMS-90166:'Missing Code Signing Entitlements.No entitlements found in boundle 'com.xxx.xxx'for executable 'xxx/xxx.app/xxx'.'，好吧，你是大爷，你说错咱改还不行吗。于是继续找google君找答案，找了[几个类似的](http://stackoverflow.com/questions/29684966/missing-code-signing-entitlements-for-resource-bundle-xcode-6-3)但是都跟我遇到的情况都不一样，而且项目急需上架，于是我准备先用前面提到的方法1：Archive打包，但是打包完成后提交还是遇到报这个错,到这里我有点懵逼了，这时候一个叫柠檬的同事说遇到过相同的情况，咨询之，对方说他们改了脚本用Archive打包就搞定了，一语惊醒梦中人(谢谢柠檬同学)，对啊哥怎么没想到可以改用脚本Archive打包呢这样又可以规避一些Xcode的bug，于是把脚本中的.app打包方式改成Archive打包后就提交成功了。详细脚本请看上面
