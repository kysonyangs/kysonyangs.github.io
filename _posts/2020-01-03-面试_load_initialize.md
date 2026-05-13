---
layout: post
title: "load"
date: 2020-01-03 18:37:52 +0800
tags: ["Xcode"]
---

#### 如何快速列出App的所有 +load 方法

控制台输入
```
br s -r "\+\[.+load\]$"
```
然后输入
```
br list
```
