---
layout: post
title: "SwiftLint 使用"
date: 2019-08-11 15:32:33 +0800
tags: ["工具"]
---

# SwiftLint
[官方文档](https://github.com/realm/SwiftLint/blob/master/README_CN.md)

<!-- TOC -->

- [SwiftLint 介绍](#swiftlint-%e4%bb%8b%e7%bb%8d)
- [SwiftLint 安装](#swiftlint-%e5%ae%89%e8%a3%85)
- [SwiftLint 规则](#swiftlint-%e8%a7%84%e5%88%99)
- [自定义配置](#%e8%87%aa%e5%ae%9a%e4%b9%89%e9%85%8d%e7%bd%ae)

<!-- /TOC -->

## SwiftLint 介绍
* [SwiftLint](https://github.com/realm/SwiftLint) 是 `Realm` 推出的一款 `Swift` 代码规范检查工具, `SwiftLint` 基于 [GitHub 公布的 Swift 代码规范](https://github.com/Artwalk/swift-style-guide/blob/master/README_CN.md) 进行代码检查，并且能够很好的和 Xcode 整合
* 配置好所有的设置之后，在 `Xcode` 中执行编译时，`SwiftLint` 会自动运行检查，不符合规范的代码会通过 `警告` 或者 `红色错误` 的形式指示出来
* 支持自定义规则,可禁用或者开启某一些规则

<!-- more -->

## SwiftLint 安装
1. 安装
  * 使用Homebrew `brew install swiftlint`
  * CocoaPods 安装  `pod 'SwiftLint'`
  * 注意在 `.gitignore` 添加 `对应目录` 避免不必要的麻烦

2. 配置
  1. `Build Phases` -> `New Run Script Phase` (-> 重命名为 SwiftLint)
  2. 添加脚本
    * Homebrew 安装
    ```
    if which swiftlint >/dev/null; then
    swiftlint
    else
    echo "warning: SwiftLint not installed, download from https://github.com/realm/ SwiftLint"
    fi
    ```
    * CocoaPods安装 `"${PODS_ROOT}/SwiftLint/swiftlint"`
      1. `Command+B` 编译项目，可以查看你不规范的地方了

## SwiftLint 规则

1. [Rules.md](https://github.com/realm/SwiftLint/blob/master/Rules.md)
2. 在代码中关闭某个规则
```
可以通过在一个源文件中定义一个如下格式的注释来关闭某个规则：
// swiftlint:disable <rule>
在该文件结束之前或者在定义如下格式的匹配注释之前，这条规则都会被禁用：
// swiftlint:enable <rule>
```
```
也可以通过添加 :previous, :this 或者 :next 来使关闭或者打开某条规则的命令分别应用于前一行，当前或者后一行代码。
// swiftlint:disable:next force_cast
let noWarning = NSNumber() as! Int
let hasWarning = NSNumber() as! Int
let noWarning2 = NSNumber() as! Int // swiftlint:disable:this force_cast
let noWarning3 = NSNumber() as! Int
// swiftlint:disable:previous force_cast
```

## 自定义配置

当编译后发现 999+警告 999+ 错误，是不是很崩溃，如果觉得规范太严格，我们可以自定义一些配置来符合自己的风格，再做一些配置忽略 `CocoaPods`、`Carthage`、`自己导入的包` 等包管理器引入的第三方库

1. 创建配置文件
    1. 根目录下 `touch .swiftlint.yml` 新建一个 `.swiftlint.yml` 的配置文件
2. 几个配置选项
```
disabled_rules: # 禁用指定的规则
  - colon
  - comma
  - control_statement
opt_in_rules: # 启用指定的规则
  - empty_count
  - missing_docs
  # 可以通过执行如下指令来查找所有可用的规则:
  # swiftlint rules
included: # 执行 linting 时包含的路径。如果出现这个 `--path` 会被忽略。
  - Source
excluded: # 执行 linting 时忽略的路径。 优先级比 `included` 更高。
  - Carthage
  - Pods
  - Source/ExcludedFolder
  - Source/ExcludedFile.swift

# 可配置的规则可以通过这个配置文件来自定义
# 二进制规则可以设置他们的严格程度
force_cast: warning # 隐式
force_try:
  severity: warning # 显式
# 同时有警告和错误等级的规则，可以只设置它的警告等级
# 隐式
line_length: 110
# 可以通过一个数组同时进行隐式设置
type_body_length:
  - 300 # warning
  - 400 # error
# 或者也可以同时进行显式设置
file_length:
  warning: 500
  error: 1200
# 命名规则可以设置最小长度和最大程度的警告/错误
# 此外它们也可以设置排除在外的名字
type_name:
  min_length: 4 # 只是警告
  max_length: # 警告和错误
    warning: 40
    error: 50
  excluded: iPhone # 排除某个名字
identifier_name:
  min_length: # 只有最小长度
    error: 4 # 只有错误
  excluded: # 排除某些名字
    - id
    - URL
    - GlobalAPIKey
reporter: "xcode" # 报告类型 (xcode, json, csv, checkstyle, junit, html, emoji)
```
3. 我的配置文件
```
disabled_rules: # rule identifiers to exclude from running
  - missing_docs
  - unused_closure_parameter
  - cyclomatic_complexity

opt_in_rules: # some rules are only opt-in
  - empty_count
# Find all the available rules by running:
# swiftlint rules

included: # paths to include during linting. `--path` is ignored if present.
  - xxx
  - xxx
  - xxx

excluded: # paths to ignore during linting. Takes precedence over `included`.
  - Pods
  - xxx
  - xxx

# configurable rules can be customized from this configuration file
# binary rules can set their severity level
force_cast: warning # implicitly
force_try:
  severity: warning # explicitly

# rules that have both warning and error levels, can set just the warning level
# implicitly
line_length: 300

function_body_length:
  - 300 # warning
  - 400 # error

function_parameter_count:
  - 10 # warning
  - 15 # error

large_tuple:
  - 6 # warning
  - 12 # error

# they can set both implicitly with an array
type_body_length:
  - 300 # warning
  - 400 # error

# or they can set both explicitly
file_length:
  warning: 1000
  error: 2000

cyclomatic_complexity:
  - 15 # warning
  - 30 # error

# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
type_name:
  min_length: 3 # only warning
  max_length: # warning and error
    warning: 40
    error: 50
  excluded: # excluded via string
    - T
    - E

identifier_name:
  min_length: # only min_length
    error: 3 # only error
  excluded: # excluded via string array
    - vc
    - id
    - in
    - kf
    - x
    - y
```