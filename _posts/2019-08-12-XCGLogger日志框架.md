---
layout: post
title: "XCGLogger 使用"
date: 2019-08-12 16:10:05 +0800
tags: ["工具"]
---

<!-- TOC -->

- [XCGLogger的介绍](#xcglogger的介绍)
- [XCGLogger的安装与配置](#xcglogger的安装与配置)
- [基本用法](#基本用法)
    - [XCGLogger初始化](#xcglogger初始化)
    - [日志输出](#日志输出)
    - [运行效果](#运行效果)
- [更高级的初始化方法](#更高级的初始化方法)
- [过滤日志信息](#过滤日志信息)
    - [按文件名过滤](#按文件名过滤)
    - [按标签过滤](#按标签过滤)
- [选择性的执行代码](#选择性的执行代码)
- [日期格式化](#日期格式化)
- [自动切换配置](#自动切换配置)
- [在后台进行日志处理](#在后台进行日志处理)
    - [下面两种方法都是实现将日志文件输出放在后台线程中进行：](#下面两种方法都是实现将日志文件输出放在后台线程中进行)
    - [也可以通过build flags来实现程序在调试、生产自动使用不同线程](#也可以通过build-flags来实现程序在调试生产自动使用不同线程)
- [实现日志文件的增量记录](#实现日志文件的增量记录)
- [实现日志文件的转储](#实现日志文件的转储)

<!-- /TOC -->

## XCGLogger的介绍

* `XCGLogger` 是一个 `debug` 日志框架，可用于 `Swift` 项目中
* 使用 `XCGLogger` ，除了可以将日志详细信息输出到控制器台外，还可以输出到指定的文件中去
* 虽然使用起来同 `NSLog()` 或 `print()` 差不多，但 `XCGLogger` 会附带更多的额外信息，比如：日期、函数名、文件名和行号

<!-- more -->

## XCGLogger的安装与配置

* 从 [`GitHub`](https://github.com/DaveWoodCom/XCGLogger) 下载最新的代码, 并拖入项目
* 或者使用 `CocoaPods` 或 `Carthage`

## 基本用法

### XCGLogger初始化

1. 定义一个全局的 `XCGLogger` 日志对象，并进行初始化
  * 选择打印所有的日志信息（级别、方法名、文件名、行号）
  * 日志除了会打印在控制台中外，还会同步输出到文件中（Library/Caches/log.txt）
  * 同时日志在控制台打印和文件输出的消息级别一样，都是 `debug` 级

  ```
  import XCGLogger

  let XCCacheDirectory: URL = {
      let urls = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask)
      return urls[urls.endIndex - 1]
  }()

  let log: XCGLogger = {
      let log = XCGLogger.default

      // 日志文件地址
      let logPath: URL = XCCacheDirectory.appendingPathComponent("XCGLogger_Log.txt")
      log.setup(level: .debug, showThreadName: true, showLevel: true, showFileNames: true, showLineNumbers: true, writeToFile: logPath)

      return log
  }()
  ```

### 日志输出

通过 `XCGLogger`内置的方法，我们可以输出各种消息级别的日志信息。比如下面样例我们打印从低到高各种级别的日志

```
log.verbose("一条verbose级别消息：程序执行时最详细的信息。")
log.debug("一条debug级别消息：用于代码调试。")
log.info("一条info级别消息：常用与用户在console.app中查看。")
log.warning("一条warning级别消息：警告消息，表示一个可能的错误。")
log.error("一条error级别消息：表示产生了一个可恢复的错误，用于告知发生了什么事情。")
log.severe("一条severe error级别消息：表示产生了一个严重错误。程序可能很快会奔溃。")
```

### 运行效果

由于我们前面将日志显示级别设置成 `debug`，所以 `verbose` 级别的日志消息就不会打印

## 更高级的初始化方法

通常情况下我们只需要像上面样例一样，使用 `setup` 方法初始化 `XCGLogger`对象相关配置即可。方便简单，代码也少。

当然 `XCGLogger` 也支持更加灵活的配置，使得我们日志记录会更加可控。比如：想让控制台显示的信息与日志文件里记录的信息不一样。

```
// 创建一个logger对象
let log = XCGLogger(identifier: "advancedLogger", includeDefaultDestinations: false)

// 控制台输出
let systemDestination = AppleSystemLogDestination(identifier: "advancedLogger.systemDestination")

// 设置控制台输出的各个配置项
systemDestination.outputLevel = .debug
systemDestination.showLogIdentifier = false
systemDestination.showFunctionName = true
systemDestination.showThreadName = true
systemDestination.showLevel = true
systemDestination.showFileName = true
systemDestination.showLineNumber = true
systemDestination.showDate = true

// logger对象中添加控制台输出
log.add(destination: systemDestination)

// 日志文件地址
let logPath: URL = XCCacheDirectory.appendingPathComponent("XCGLogger_Log.txt")
let fileDestination = FileDestination(writeToFile: logPath, identifier: "advancedLogger.fileDestination")

// 设置文件输出的各个配置项
fileDestination.outputLevel = .debug
fileDestination.showLogIdentifier = false
fileDestination.showFunctionName = true
fileDestination.showThreadName = true
fileDestination.showLevel = true
fileDestination.showFileName = true
fileDestination.showLineNumber = true
fileDestination.showDate = true

// Xcode8以上无效了
// 设置输出语句颜色和样式（搭配XcodeColors使用）
/**
let ansiColorLogFormatter: ANSIColorLogFormatter = ANSIColorLogFormatter()
ansiColorLogFormatter.colorize(level: .verbose, with: .colorIndex(number: 244), options: [.faint])
ansiColorLogFormatter.colorize(level: .debug, with: .black)
ansiColorLogFormatter.colorize(level: .info, with: .blue, options: [.underline])
ansiColorLogFormatter.colorize(level: .warning, with: .red, options: [.faint])
ansiColorLogFormatter.colorize(level: .error, with: .red, options: [.bold])
ansiColorLogFormatter.colorize(level: .severe, with: .white, on: .red)
fileDestination.formatters = [ansiColorLogFormatter]
*/

// 文件输出在后台处理
fileDestination.logQueue = XCGLogger.logQueue

// logger对象中添加控制台输出
log.add(destination: fileDestination)

// 开始启用
log.logAppDetails()

// 更改权限提示 eg: [debug]=>"🔹"
let emojiLogFormatter = PrePostFixLogFormatter()
emojiLogFormatter.apply(prefix: "🗯🗯🗯 ", postfix: " 🗯🗯🗯", to: .verbose)
emojiLogFormatter.apply(prefix: "🔹🔹🔹 ", postfix: " 🔹🔹🔹", to: .debug)
emojiLogFormatter.apply(prefix: "ℹ️ℹ️ℹ️ ", postfix: " ℹ️ℹ️ℹ️", to: .info)
emojiLogFormatter.apply(prefix: "⚠️⚠️⚠️ ", postfix: " ⚠️⚠️⚠️", to: .warning)
emojiLogFormatter.apply(prefix: "‼️‼️‼️ ", postfix: " ‼️‼️‼️", to: .error)
emojiLogFormatter.apply(prefix: "💣💣💣 ", postfix: " 💣💣💣", to: .severe)
log.formatters = [emojiLogFormatter]
```

## 过滤日志信息

### 按文件名过滤

1. 下面代码不显示 `AppDelegate.swift` 中输出的日志，其它文件日志都会显示
  ```
  log.filters = [FileNameFilter(excludeFrom: ["AppDelegate.swift"],   excludePathWhenMatching: true)]
  ```

2. 下面代码只显示 AppDelegate.swift 中输出的日志，其它文件日志都不显示
  ```
  log.filters = [FileNameFilter(includeFrom: ["AppDelegate.swift"],   excludePathWhenMatching: true)]
  ```

### 按标签过滤
1. 使用自定义的标签字符串

  比如我们在输出一条 debug 消息时，附上一个自定义的标签（sensitive）
  ```
  let sensitiveTag = XCGLogger.Constants.userInfoKeyTags
  log.debug("这里进行用户身份验证。", userInfo: [tags: "sensitive"])
  ```
  下面设置分别是不显示这个标签的日志，以及只显示这个标签的日志
  ```
  // 下面代码不显示标签为"sensitive"的日志，其它日志都会显示
  log.filters = [TagFilter(excludeFrom: ["sensitive"])]

  // 下面代码只显示标签为"sensitive"的日志，其它日志都不会显示
  log.filters = [TagFilter(includeFrom: ["sensitive"])]
  ```

2. 通过扩展 `Tag` 和 `Dev` 这两个标签结构体
  `Tag` 和 `Dev` 是 `XCGLogger` 自带的，我们这里通过扩展，在其上面添加一个些自定义标签。
  ```
  extension Tag {
      static let sensitive = Tag("sensitive")
      static let ui = Tag("ui")
      static let data = Tag("data")
  }

  extension Dev {
      static let dave = Dev("dave")
      static let sabby = Dev("sabby")
  }
  ```
  我们日志输出时可以混合使用多个标签：
  ```
  // 设置多个标签
  log.debug("A tagged log message", userInfo: Dev.dave | Tag.sensitive)

  // 设置单个标签
  log.debug("Another tagged log message", userInfo: Dev.dave.dictionary)
  ```
  下面根据便签进行日志过滤：
  ```
  // 下面代码不显示标签只为"dave"或"sensitive"的日志，其它日志都会显示
  log.filters = [TagFilter(excludeFrom: [Dev.dave, Tag.sensitive])]

  // 下面代码只显示标签为"dave"或"sensitive"的日志，其它日志都不会显示
  log.filters = [TagFilter(includeFrom: [Dev.dave, Tag.sensitive])]
  ```

## 选择性的执行代码

有时我们日志输出前要进行一些计算或操作，再将结果通过日志打印出来。由于这些操作与程序实际运行无关，只是用于日志打印，那么我们可以将其放置在日志方法闭包中。这样既保证代码的干净，同时如果日志不输出的话也不会浪费系统资源。

比如下面样例，如果我们将日志消息级别调到 `debug`以上，那么下面日志不会输出，且内部的循环操作自然也不会执行。

```
log.debug {
    var total = 0.0
    for receipt in receipts {
        total += receipt.total
    }

    return "Total of all receipts: \(total)"
}
```

## 日期格式化
默认日志输出的格式是 "2000-01-01 00:00:00.000" 这种形式。当然我们也可以改成任意的自定义的格式。
```
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "MM/dd/yyyy hh:mma"
dateFormatter.locale = Locale.current
log.dateFormatter = dateFormatter
```

## 自动切换配置

通过使用 `Swift` 的编译标志（`build flags`）,我们可以在程序调试、生产版本中自动使用不同的日志级别和日志条目。
```
#if DEBUG
log.setup(level: .debug, showThreadName: true, showLevel: true,
          showFileNames: true, showLineNumbers: true)
#else
log.setup(level: .severe, showThreadName: true, showLevel: true,
          showFileNames: true, showLineNumbers: true)
#endif
```

## 在后台进行日志处理

默认情况下，日志输出是在调用它们的进程里进行的。这样可以确保日志消息可以立刻显示，方便我们使用断点调试时可以立刻看到结果。

但在发布的生产版本中，如果日志也在当前线程中执行会对性能造成一些影响。我们可以指定日志目标进程使用哪个执行队列中。

### 下面两种方法都是实现将日志文件输出放在后台线程中进行：

```
//方式1
fileDestination.logQueue = XCGLogger.logQueue

//方式2
fileDestination.logQueue = DispatchQueue.global(qos: .background)
```

### 也可以通过build flags来实现程序在调试、生产自动使用不同线程
```
#if DEBUG
    log.setup(level: .debug, showThreadName: true, showLevel: true,
              showFileNames: true, showLineNumbers: true)
#else
    log.setup(level: .severe, showThreadName: true, showLevel: true,
              showFileNames: true, showLineNumbers: true)
    if let consoleLog = log.logDestination(XCGLogger.Constants.baseConsoleDestinationIdentifier)
        as? ConsoleDestination {
        consoleLog.logQueue = XCGLogger.logQueue
    }
#endif
```

## 实现日志文件的增量记录

默认情况下，日志文件的输出的是覆盖模式的。也就是说，我们指定了一个日志文件路径，程序每次重新启动时，如果这个文件原来就存在，其内容也会被覆盖。

* 我们可以在日志配置时将 `shouldAppend` 参数设置为 `true`，这样如果原来的文件存在，也不会重新覆盖。而是将新日志信息添加到文件尾部。
*
* 同时 `appendMarker`可以设置个字符串，其在每次程序启动时会自动添加到日志文件中。方便我们区分开每次程序运行的日志。
*
```
...
//文件出输出
let fileDestination = FileDestination(writeToFile: logURL,
                                      identifier: "advancedLogger.fileDestination",
                                      shouldAppend: true,
                                      appendMarker: "-- Relauched App --")
...
```

## 实现日志文件的转储

上面的样例中，我们都是将日志记录到 Caches 文件夹下的 log.txt。如果是增量日志的话，那个这个文件就会越来越大。所以我们可以通过 rotateFile(to:) 方法实现日志的转储。比如：每隔一段时间，或每当日志文件到一定大小时调用下该方法，将日志内容转储到另一个文件中。（注意：转储后原来日志文件里的内容会清空。）

具体转储操作的步骤如下：
* 首先通过 `XCGLogger` 的 `destination(withIdentifier:)` 方法获取 FileDestination 对象。
* 再调用 `FileDestination` 对象的 `rotateFile(to:)` 方法将日志文件转储到指定路径。
* `rotateFile(to:)` 方法执行成功后会返回 `true`。根据结果我们可以进行一些后续操作。比如：压缩、邮件发送日志文件等等。

下面代码我们将日志文件转储到用户文档目录下：
```
// 获取FileDestination
let fileDestination = log.destination(withIdentifier: "advancedLogger.fileDestination")
    as! FileDestination

// 获取用户文档目录
let documentPath = FileManager.default.urls(for: .documentDirectory,
                                         in: .userDomainMask)[0]
let logURL = documentPath.appendingPathComponent("log.txt")

// 将当前的日志文件复制到用户文档目录中去
fileDestination.rotateFile(to: logURL)
```
