---
layout: post
title: "CocoaPods/Carthage"
date: 2017-12-05 15:21:48 +0800
tags: ["CocoaPods"]
---

* `CocoaPods` (默认)自动建立和更新一个 `Xcode workspace`，用来管理你的项目和所有依赖。`Carthage` 使用 `xcodebuild` 来编译出二进制库，剩下的集成工作完全交给开发人员。
* `CocoaPods` 使用起来方便， `Carthage` 更加灵活并且对现有项目没有太多的侵略性。
* `CocoaPods` 希望建立一个生态系统，可以更加方便的发现和集成第三方代码库。 `Carthage` 希望变成一个去中心化的依赖管理系统，不提供中心化的项目列表，减少维护成本和单点失败的概率。不过这样给开发人员寻找项目带来不便。
* `CocoaPods` 的项目需要配置 `podspec` 文件，包含了项目和第三方库的信息。 `Carthage` 并不使用类似的配置文件，第三方库的依赖关系是通过 Xcode 项目来配置的。

<!-- more -->
### CocoaPods 安装与使用

#### 安装

1. 安装需要用到 Ruby，但是 Mac 自带了 Ruby，如果嫌弃版本过低，可以更新一下 ruby。
    a. 安装 RVM，`curl -L https://get.rvm.io | bash -s stable`
    b. 查看 rvm 版本，`rvm -v`
    c. rvm 安装 ruby
        a. 列出已知的ruby版本，`rvm list known`
        b. 选择版本安装，`rvm install 2.0.0`
        c. 查询已经安装的 ruby，`rvm list`
        d. 卸载一个已安装版本，`rvm remove 1.9.2`
    d. 设置默认 ruby 版本，`rvm 2.0.0 --default`
2. 安装 CocoaPods
```
sudo gem install cocoapods
```

然后你会发现无比的慢，但是我们可以通过换源来提高那么一点速度
a. `gem sources -l` 查看现有的源
b. `gem sources --remove https://rubygems.org/` 删除原来的源
c. `gem sources -a http://ruby.taobao.org/` 更换为淘宝的源

#### 使用

example: 使用CocoaPods 导入 AFNetworking 库

1. 在项目(CocosPodsDemo)地址下，新建一个文件 podfile, 基本使用如下：
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'

target "CocosPodsDemo" do

pod 'AFNetworking'

end
```
2. 项目路径下使用 `pod install`
3. 完成后打开 `CocosPodsDemo.xcworkspace` 完成项目开发.
4. 更多使用方法自行`百度`

### Carthage 安装与使用

#### 安装
```
brew update
brew install carthage
```
**注意**
- 如果报错提示需要安装 Xcode 插件，`xcode-select --install`

查看版本
```
carthage version
```

#### 使用

example: 使用 Carthage 导入 AFNetworking 库

1. 在项目(CarthageDemo)地址下，新建一个文件 Cartfile, 基本使用如下：
```
github "AFNetworking/AFNetworking" ~> 3.0
```
2. 在项目(CarthageDemo)地址下，执行命令
```
carthage update --platform iOS
```
3. 打开 Carthage 查看生成的文件目录
```
open Carthage
```
4. 配置项目
    a. 打开项目，点击 `Target -> Build Phases -> Link Library with Libraries` 选择 `Carthage/Build` 目录中的 `framework`
    b. 添加编译的脚本（该脚本文件保证在提交归档时会对相关文件和dSYMs进行复制），点击 `Target -> Build Phases -> + -> New Run Script Phase`
    c. 添加脚本，`/usr/local/bin/carthage copy-frameworks`
    d. 添加 `"Input Files"` `$(SRCROOT)/Carthage/Build/iOS/AFNetworking.framework`


