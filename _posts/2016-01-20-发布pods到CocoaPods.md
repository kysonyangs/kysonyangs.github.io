---
layout: post
title: "发布pods到CocoaPods"
date: 2016-01-20 15:12:36 +0800
tags: ["CocoaPods"]
---

### 注册
* `pod trunk register  邮箱 '用户名' --description='电脑描述'`
* `pod trunk register xxx@gmail.com 'KL' --verbose`

<!-- more -->

### 查收邮件
* 如果是QQ邮箱，可能会被放到“垃圾箱”中，并不一定是“收件箱”
* 点击邮件中的链接：
https://trunk.cocoapods.org/sessions/verify/xxxx

### 接下来查看个人信息
* `pod trunk me`

```
  - Name:     xxx
  - Email:    xxxxxx@qq.com
  - Since:    January 28th, 03:53
  - Pods:     None
  - Sessions:
    - January 28th, 04:28 - June 5th, 04:34. IP: xxx.xxx.xxx.xxx Description: Macbook Pro
```
* 中间可能遇到这种错误

```
NoMethodError - undefined method 'last' for #<Netrc::Entry:0x007fc59c246378>
```
* 这时候需要尝试更新gem源或者pod
    * `sudo gem update --system`
    * `sudo gem install cocoapods`
    * `sudo gem install cocospods-trunk`

### 创建podspec文件
* 接下来需要在项目根路径创建一个podspec文件来描述你的项目信息
    * `pod spec cretae 文件名`
    * 比如pod spec cretae KLTest 就会生成一个KLTest.podspec

### 填写podspec内容
```
Pod::Spec.new do |s|
  s.name         = "KLTest"
  s.version      = "0.0.1"
  s.summary      = "The fastest and most convenient conversion between JSON and model"
  s.homepage     = "https://github.com/xxx/KLTest"
  s.license      = "MIT"
  s.author             = { "KL" => "xxxxx@qq.com" }
  s.social_media_url   = "http://weibo.com/exceptions"
  s.source       = { :git => "https://github.com/xxx/KLTest.git", :tag => s.version }
  s.source_files  = "KLTestExample/KLTestExample/KLTest"
  s.requires_arc = true
end
```
* 值得注意的是，现在的podspec必须有tag，所以最好先打个tag，传到github
    * `git tag 0.0.1`
    * `git push --tags`

### 检测podspec语法
* `pod spec lint KLTest.podspec`
* `pod spec lint KLTest.podspec —allow-warnings` 忽略所有警告

### 发布podspec
* `pod trunk push KLTest.podspec`
* 如果是第一次发布pod，需要去https://trunk.cocoapods.org/claims/new认领pod

### 检测
* `pod setup` : 初始化
* `pod repo update` : 更新仓库
* `pod search MJExtension`

### 仓库更新
* 如果仓库更新慢，可以考虑更换仓库镜像
    * `pod repo remove master`
    * `pod repo add master http://git.oschina.net/akuandev/Specs.git`