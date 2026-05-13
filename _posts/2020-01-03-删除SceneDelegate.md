---
layout: post
title: "新建项目删除SceneDelegate"
date: 2020-01-03 18:37:52 +0800
tags: ["Xcode"]
---

因为Xcode11之后新创建的工程会多出两个文件 SceneDelegate ，所以如果需要使用旧版的话需要自己增删一些文件/代码。

1. 删除 `SceneDelegate.h/.m/.swift` 文件
2. Info.plist 文件 删除 `Application Scene Manifest`
3. AppDelegate 里面删除 `UISceneSession lifecycle 下的代码`
4. 添加 `@property (strong, nonatomic) UIWindow *window;`
5. 不用sb的话，AppDelegate 添加创建window的代码.