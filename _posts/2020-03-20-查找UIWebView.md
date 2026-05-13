---
layout: post
title: "项目中查找UIWebView使用"
date: 2020-03-20 20:12:36 +0800
tags: ["iOS"]
---

Apple已弃用 `UIWebView` 并且 `UIWebView` 从 **2020年4月** 开始不接受新应用，从 2020年UIWebView 12月开始不接受应用更新。如果应用使用UIWebView，则应将其替换为WKWebView。

UIWebView从应用程序中删除很容易，但是如果使用了一些第三方库，它们也可能包含UIWebView。需要找到所有这些文件并对其进行更新（如果有），或者进行替换。这个过程并不简单。

如果第三方库用作代码（例如通过）Cocoapods，则可以在其来源中进行文本搜索UIWebView。

```
grep -r 'UIWebView' .
```

如果第三方库是 `.framework` 不带源文件的文件，则有另一种检查方法UIWebView。

可以使用 `nm` 来获取可执行文件的符号表中 `.framework`

```
nm AWSDK.framework/AWSDK | grep -i UIWebView
```

