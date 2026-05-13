---
layout: post
title: "URLNavigator 使用"
date: 2019-08-11 15:32:33 +0800
tags: ["工具"]
---

# URLNavigator 框架

> ⛵️ URLNavigator 是 Swift 下一个优雅的 URL 路由。它提供了通过 URL 导航到 view controller 的方式。URL 参数的对应关系通过 `URLNavigator.register(_:_:)` 方法进行设置。
>
> URLNavigator 提供了两种方法来设置 URL 参数的对应关系：URLNavigable 和 URLOpenHandler。URLNavigable 通过自定义的初始化方法进行设置，URLOpenHandler 通过一个可执行的闭包进行设置。初始化方法和闭包都接受一个 `URL` 和`占位符`值。

## 开始

### 1. 理解 URL 模式

URL 模式可以包含多个占位符。占位符将会被匹配的 URL 中的值替换。使用 `<` 和 `>` 来设置占位符。占位符的类型可以设置为：`string(默认)`, `int`, `float`, 和 `path`。

例如，`myapp://user/<int:id>` 将会和下面的 URL 匹配：

- `myapp://user/123`
- `myapp://user/87`

但是，无法和下面的 URL 配置：

- `myapp://user/devxoul` (类型错误，需要 int)
- `myapp://user/123/posts` (url 的结构不匹配))
- `/user/devxoul` (丢失 scheme)

<!-- more -->

### 2. View Controllers 和 URL 打开操作的匹配

URLNavigator 通过 URL 模式来匹配 view controllers 和 URL 的打开操作。下面是使用 view controller 和闭包映射 URL 模式的示例。每个闭包有三个参数：`url`, `values` 和 `context`。

- `url` 是通过 `push()` 或者 `present()` 传递的 URL 参数.
- `values` 是一个包含 URL 占位符的 `keys` 和 `values` 的字典.
- `context` 是通过 `push()`, `present()` 或 `open()` 传递的额外值的字典。

```
let navigator = Navigator()

// register view controllers
navigator.register("myapp://user/<int:id>") { url, values, context in
  guard let userID = values["id"] as? Int else { return nil }
  return UserViewController(userID: userID)
}
navigator.register("myapp://post/<title>") { url, values, context in
  return storyboard.instantiateViewController(withIdentifier: "PostViewController")
}

// register url open handlers
navigator.handle("myapp://alert") { url, values, context in
  let title = url.queryParameters["title"]
  let message = url.queryParameters["message"]
  presentAlertController(title: title, message: message)
  return true
}
```

### 3. 弹出方式（Pushing, Presenting）与操作 URLs

URLNavigator 可以通过执行一个带 URLs 参数的闭包来 push 或者 presenet 对应的 view controllers。

在 `push()` 方法中，通过指定 `from` 参数可以设置弹出新 `view controller` 的 `navigation controller`。同样，在 `present()` 方法中，通过指定 `from` 参数可以设置弹出新 `view controller` 的 `view controller`。如果设置为 `nil`，也就是**默认值**，当前应用程序最顶层的 `view controller` 将会被用来 `push` 或 `present` 新的 `view controller`。

`present()` 还有一个额外的参数 `wrap`。如果将这个参数设置为 `UINavigationController` 类型，则会用这种类型的导航控制器把要弹出的新的 `view controller` 包住。这个参数的默认值为 **`nil`**。

```
Navigator.push("myapp://user/123")
Navigator.present("myapp://post/54321", wrap: UINavigationController.self)

Navigator.open("myapp://alert?title=Hello&message=World")
```
