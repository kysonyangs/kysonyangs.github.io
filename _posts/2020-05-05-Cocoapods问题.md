---
layout: post
title: "CocoaPods问题"
date: 2020-05-05 19:21:48 +0800
tags: ["CocoaPods"]
---

## CDN: trunk URL couldn't be downloaded:

由于 CocoaPods 1.8 将 CDN 切换为默认的 spec repo 源，并附带一些增强功能！CDN 支持最初是在 1.7 版本中引入的，最终在 1.7.2 中完成。 它旨在大大加快初始设置和依赖性分析。但是无法连接外网的原因，所以 pod install/update 一直报错.

按照官方文档 podfile 文件中添加 source 源：

```
source 'https://github.com/CocoaPods/Specs.git'
```

删除 trunk 源

```
pod repo remove trunk
```

<!-- more -->