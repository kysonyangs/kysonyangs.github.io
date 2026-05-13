---
layout: post
title: "SwiftLint 规则收录"
date: 2019-08-11 19:32:33 +0800
tags: ["工具"]
---

规则目录，按字母排序排列

#### anyobject_protocol

对纯类协议使用 `AnyObject` 而不是 `class`

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- |
| anyobject_protocol | 未启用 | yes | lint |

示例

```
// 推荐
protocol SomeProtocol {}
protocol SomeClassOnlyProtocol: AnyObject {}
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {}
@objc protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {}

// 不推荐，触发警告
protocol SomeClassOnlyProtocol: class {}
protocol SomeClassOnlyProtocol: class, SomeInheritedProtocol {}
@objc protocol SomeClassOnlyProtocol: class, SomeInheritedProtocol {}
```

<!-- more -->

#### array_init

序列转化成数组时, 优先使用数组转化, 而不是seq.map{$0} 将序列转换为数组

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- |
| array_init | 未启用 | no | lint |

示例

```
// 推荐
Array(foo)
foo.map { $0.0 }
foo.map { $1 }
foo.map { $0() }
foo.map { ((), $0) }
foo.map { $0! }
foo.map { $0! /* force unwrap */ }
foo.something { RouteMapper.map($0) }
foo.map { !$0 }
foo.map { /* a comment */ !$0 }

// 不推荐，触发警告
foo.map({ $0 })
foo.map { $0 }
foo.map { return $0 }
foo.map { elem in
   elem
}
foo.map { elem in
   return elem
}
foo.map { (elem: String) in
   elem
}
foo.map { elem -> String in
   elem
}
foo.map { $0 /* a comment */ }
foo.map { /* a comment */ $0 }
```

#### attributes

函数和类型中，属性应该在一行

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- |
| attributes | 未启用 | no | style |

示例

```
// 推荐
@objc var x: String
@IBOutlet private var label: UILabel

// 不推荐，触发警告
@objc
var x: String
```

#### block_based_kvo

使用Swift 3.2+时，首选系统的KVO 的API和keypath

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- | --- |
| block_based_kvo | 启用 |no | idiomatic |

示例

```
// 推荐
let observer = foo.observe(\.value, options: [.new]) { (foo, change) in
   print(change.newValue)
}

// 不推荐，触发警告
class Foo: NSObject {
   override func observeValue(forKeyPath keyPath: String?, of object: Any?,
                               change: [NSKeyValueChangeKey : Any]?,
                               context: UnsafeMutableRawPointer?) {}
}
```

#### class_delegate_protocol

委托协议应该仅是class，可以被弱引用

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| class_delegate_protocol | 启用 | no | lint |

示例

```
// 推荐
protocol FooDelegate: class {}
protocol FooDelegate: class, BarDelegate {}
protocol Foo {}
class FooDelegate {}
@objc protocol FooDelegate {}
@objc(MyFooDelegate)
protocol FooDelegate {}
protocol FooDelegate: BarDelegate {}
protocol FooDelegate: AnyObject {}
protocol FooDelegate: NSObjectProtocol {}

// 不推荐，触发警告
protocol FooDelegate {}
protocol FooDelegate: Bar {}
```

#### Closing Brace Spacing

小括号包含大括号的话之间不能有空格

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| closing_brace | 启用 | yes | style |

示例

```
// 推荐
[1, 2].map({ $0 })
[1, 2].map(
  { $0 }
)

// 不推荐，触发警告
[1, 2].map({ $0 } )
[1, 2].map({ $0 }   )
[1, 2].map( { $0 })
```

#### Closure Body Length

必报的函数体不应该太多行,**20行警告，100行报错**

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- |
| closure_body_length | 未启用 | no | metrics |

#### Closure End Indentation

闭包的封闭端和开始端有相同的缩进

就是 大括号（一般是方法）上下对齐的问题，这样使code看起来更加整洁

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| closure_end_indentation | 未启用 | no | style |

示例

```
// 推荐
[1, 2].map { $0 + 1 }
SignalProducer(values: [1, 2, 3])
   .startWithNext { number in
       print(number)
   }
function {
    ..........
}

// 不推荐，触发警告
SignalProducer(values: [1, 2, 3])
   .startWithNext { number in
       print(number)
}
function {
    ..........
    }
```

#### Closure Parameter Position

闭包参数位置，闭包参数应该和大括号左边在同一行

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| closure_parameter_position | 启用 | no | style |

示例

```
// 推荐
let names = [1, 2, 3]
names.forEach { (number) in
    print(number)
}

let names = [1, 2, 3]
names.map { number in
    number + 1
}

// 不推荐，触发警告
let names = [1, 2, 3]
names.forEach {
    (number) in
    print(number)
}

let names = [1, 2, 3]
names.map {
    number in
    number + 1
}
```

#### Closure Spacing

在闭包的{}中间要有一个空格,如map({ $0 })

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| closure_spacing | 未启用 | no | style |

示例

```
// 推荐
map({ $0 })
[].map ({ $0.description })

// 不推荐，触发警告
map({$0 })
map({ $0})
map({$0})
[].map ({$0.description     })
```


#### Collection Element Alignment

集合中的所有元素应该垂直对齐

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 |
| --- | --- | --- | --- |
| collection_alignment | 未启用 | no | style |

示例

```
// 推荐
doThings(arg: [
    "foo": 1,
    "bar": 2,
    "fizz": 2,
    "buzz": 2
])
let abc = [
    "alpha": "a",
    "beta": "b",
    "gamma": "g",
    "delta": "d",
    "epsilon": "e"
]
let abc = [1, 2, 3, 4]

// 不推荐，触发警告
let abc = [
    "alpha": "a",
      "beta": "b",
    "gamma": "g",
    "delta": "d",
  "epsilon": "e"
]
```

#### Colon

冒号应该紧挨左边，右边空一格

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| colon | 启用 | yes | style |

示例

```
// 不会触发警告
let abc: String = "jun"
let abc = [1: [3: 2], 3: 4]
let abc = [1: [3: 2], 3: 4]

// 会触发警告
let jun:Void
let jun : Void
let jun :Void
let jun:   Void
```

#### Comma Spacing

逗号应该紧挨左边，右边空一格

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| comma | 启用 | yes | style |

示例

```
// 推荐
[a, b, c, d]

// 不推荐，触发警告
[a ,b]
```

#### Compiler Protocol Init

推荐使用自变量初始化变量

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| compiler_protocol_init | 启用 | no | lint |

示例

```
// 推荐
let set: Set<Int> = [1, 2]

// 不推荐
let set = Set(arrayLiteral: 1, 2)
let set = Set.init(arrayLiteral: 1, 2)
```

#### Conditional Returns on Newline

条件语句在下一行返回

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| conditional_returns_on_newline | 未启用 | no | style |

示例

```
// 推荐
if true {
    return
}

guard true else {
    return
}

// 不推荐, 会触发warning
if true { return }
guard true else { return }
```
#### contains_over_first_not_nil

类似first函数不能判断是否为nil

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| contains_over_first_not_nil | 未启用 | no | performance |

示例

```
// 推荐
let first = myList.first(where: { $0 % 2 == 0 })
let first = myList.first { $0 % 2 == 0 }

// 不推荐
myList.first { $0 % 2 == 0 } != nil
myList.first(where: { $0 % 2 == 0 }) != nil
```

#### control_statement

控制语句, for，while，do，catch语句中的条件不能包含在()中

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| control_statement | 启用 | no | style |

示例

```
// 推荐
if condition {
if (a, b) == (0, 1) {

// 不推荐
if (condition) {
if(condition) {
if ((a || b) && (c || d)) {
```

#### custom_rules

自定义规则。这个属性可以通过提供正则表达式来创建自定义规则，可选指定语法类型搭配，安全、级别和要陈列的什么信息。这个属性`推荐`熟悉使用正则表达式的人使用。

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| custom_rules | 启用 | no | style |

#### cyclomatic_complexity

循环复杂度。函数体的复杂度应该要限制，这个属性主要约束条件句、循环句中的循环嵌套问题，当嵌套太多的循环时，则会触发swiftlint中的warning和error，当达到10个循环嵌套时就会报warning，达到20个循环嵌套时就会报error

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| cyclomatic_complexity | 启用 | no | metrics |

#### discarded_notification_center_observer

当使用注册的通知时, 应该存储返回的观察者, 便于用完之后移除通知

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| discarded_notification_center_observer | 启用 | no | lint |

示例

```
// 推荐
let foo = nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil) { }

let foo = nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil, using: { })

func foo() -> Any {
   return nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil, using: { })
}

// 不推荐
nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil) { }

nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil, using: { })

@discardableResult func foo() -> Any {
   return nc.addObserver(forName: .NSSystemTimeZoneDidChange, object: nil, queue: nil, using: { })
}
```

#### discouraged_direct_init

阻止直接初始化导致的错误类型, 有类方法的,用类方法初始化(不建议直接init初始化)

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| discouraged_direct_init | 启用 | no | lint |

示例

```
// 推荐
let foo = UIDevice.current
let foo = Bundle.main
let foo = Bundle(path: "bar")
let foo = Bundle(identifier: "bar")

// 不推荐
let foo = UIDevice()
let foo = Bundle()
let foo = bar(bundle: Bundle(), device: UIDevice())
```

#### discouraged_optional_boolean

不建议使用可选布尔值

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| discouraged_optional_boolean | 未启用 | no | idiomatic |

示例

```
// 推荐
var foo: Bool
var foo: [String: Bool]
var foo: [Bool]
let foo: Bool = true

// 不推荐
var foo: Bool?
var foo: [String: Bool?]
var foo: [Bool?]
let foo: Bool? = nil
```

#### discouraged_object_literal

优先使用对象初始化方法, 不建议使用代码块初始化

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| discouraged_object_literal | 未启用 | no | idiomatic |

示例

```
// 不建议
let white = #colorLiteral(red: 1.0, green: 1.0, blue: 1.0, alpha: 1.0)
let image = ↓#imageLiteral(resourceName: "image.jpg")
```

#### dynamic_inline

避免一起使用 dynamic 和 @inline(_ _always)， 否则报 error

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| dynamic_inline | 启用 | no | lint |

示例

```
// 正确的做法
class LangKe {
    dynamic func myFunction() {

    }
}

class LangKe {
    @inline(__always) func myFunction() {

    }
}

class LangKe {
    @inline(never) dynamic func myFunction() {

    }
}

// 只要同时使用 dynamic 和 @inline(_ _always)都报错 error！！！
class LangKe {
    @inline(__always) public dynamic func myFunction() {

    }
}
```

#### empty_count

建议使用isEmpty判断,而不是使用count==0判断

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| empty_count | 未启用 | no | performance |

示例

```
// 推荐
    print("为空")
} else {
    print("不为空")
}

// 不推荐
let number = "long"
if number.characters.count == 0 {
    print("为空")
} else {
    print("不为空")
}
```

#### empty_enum_arguments

当枚举与关联类型匹配时，如果不使用它们，参数可以省略

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| empty_enum_arguments | 启用 | yes | style |

示例

```
// 推荐
switch foo {
    case .bar: break
}
switch foo {
    case .bar(let x): break
}
switch foo {
    case let .bar(x): break
}
switch (foo, bar) {
    case (_, _): break
}
switch foo {
    case "bar".uppercased(): break
}

// 不推荐
switch foo {
    case .bar(_): break
}
switch foo {
    case .bar(): break
}
switch foo {
    case .bar(_), .bar2(_): break
}
```

#### empty_parameters

闭包参数为空时,建议使用() ->Void, 而不是Void ->Void

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| empty_parameters | 启用 | yes | style |

示例

```
// 推荐
let abc: () -> Void

func foo(completion: () -> Void) {

}

// 不推荐，报错
let bcd: Void -> Void

func foo(completion: Void -> Void) {

}
```

#### empty_parentheses_with_trailing_closure

在使用尾随闭包的时候， 应该尽量避免使用空的圆括号

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| empty_parentheses_with_trailing_closure | 启用 | yes | style |

#### explicit_acl

所有属性和方法的声明, 都应该明确指定修饰关键字

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| explicit_acl | 未启用 | no | idiomatic |

示例

```
// 推荐
internal enum A {}
public final class B {}
private struct C {}
internal func a() { let a =  }
private struct C { let d = 5 }
internal class A { deinit {} }
internal protocol A {
    func b()
    var c: Int
}


// 不推荐
enum A {}
final class B {}
internal struct C { let d = 5 }
public struct C { let d = 5 }
func a() {}
internal let a = 0
func b() {}
```

#### explicit_type_interface

声明的属性应该明确其类型, 如: var myVar: Int = 0

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| explicit_type_interface | 未启用 | no | idomatic |

示例

```
// 推荐
class Foo {
  var myVar: Int? = 0
  let myLet: Int? = 0
}

// 不推荐
class Foo {
  var myVar = 0
  let myLet = 0
}
```

#### extension_access_modifier

在自定义类中,推荐使用extension扩展

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| extension_access_modifier | 未启用 | no | idiomatic |

#### no_extension_access_modifier

在extension扩展前面,不建议使用(fileprivate, public)等修饰符

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| no_extension_access_modifier | 未启用 | no | idiomatic |

示例

```
// 不推荐
private extension String {}
public extension String {}
open extension String {}
internal extension String {}
fileprivate extension String {}
```

#### fallthrough

switch语句中不建议使用fallthrough

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| fallthrough | 启用 | no | idiomatic |

示例

```
// 推荐
switch foo {
case .bar, .bar2, .bar3:
    something()
}

// 不推荐
switch foo {
case .bar:
    fallthrough
case .bar2:
    something()
}
```

#### fatal_error_message

执行fatalError错误时,建议有一个提示信息; 如:fatalError("Foo")

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| fatal_error_message | 未启用 | no | idiomatic |

示例

```
// 推荐
required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}

// 不推荐
required init?(coder aDecoder: NSCoder) {
    fatalError("")
}
```

#### file_header

文件头。新建的文件开始的注释应该一样

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| file_header | 未启用 | no | style |

示例

```
/// 不会触发warning
/// 如果我新建一个工程，在ViewController.swift文件中， 开始的注释应该是：

//  ViewController.swift
//  SwiftLint
//
//  Created by langke on 17/1/17.
//  Copyright © 2017年 langke. All rights reserved.
//

改变一下变为：
//
//  MyViewController.swift...................由于这里和外面的文件名不一样，所以触发warning（实际上在swift 3.0上测试这个属性暂时没有任何作用！！）
//  SwiftLint
//
//  Created by langke on 17/1/17.
//   Copyright © 2017年 langke. All rights reserved................官方terminal表示，Copyright和Created没有对齐，也会触发warning！！！
//
```

#### file_length

文件内容行数, 超过400行warning, 超过1000行给error

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| file_length | 启用 | no | metrics |

#### first_where

不建议在使用filter和map函数后直接使用.first

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| first_where | 未启用 | no | performance |

示例

```
public static let description = RuleDescription(
    identifier: "first_where",
    name: "First Where",
    description: "Prefer using `.first(where:)` over `.filter { }.first` in collections.",
    kind: .performance,

    // 不会触发警告
    nonTriggeringExamples: [
        "kinds.filter(excludingKinds.contains).isEmpty && kinds.first == .identifier\n",
        "myList.first(where: { $0 % 2 == 0 })\n",
        "match(pattern: pattern).filter { $0.first == .identifier }\n",
        "(myList.filter { $0 == 1 }.suffix(2)).first\n"
    ],
    // 以下写法会触发警告
    triggeringExamples: [
        "↓myList.filter { $0 % 2 == 0 }.first\n",
        "↓myList.filter({ $0 % 2 == 0 }).first\n",
        "↓myList.map { $0 + 1 }.filter({ $0 % 2 == 0 }).first\n",
        "↓myList.map { $0 + 1 }.filter({ $0 % 2 == 0 }).first?.something()\n",
        "↓myList.filter(someFunction).first\n",
        "↓myList.filter({ $0 % 2 == 0 })\n.first\n",
        "(↓myList.filter { $0 == 1 }).first\n"
    ]
)
```

#### for_where

在for循环中,不建议使用单个if语句或者只使用一次循环变量,可使用where或者if{}else{}语句

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| for_where | 启用 | no | idiomatic |

示例

```
// 推荐
for user in users where user.id == 1 { }
for user in users {
   if let id = user.id { }
}
for user in users {
   if var id = user.id { }
}
for user in users {
   if user.id == 1 { } else { }
}
for user in users {
   if user.id == 1 { }
   print(user)
}
for user in users {
   let id = user.id
   if id == 1 { }
}
for user in users {
   if user.id == 1 && user.age > 18 { }
}

// 不推荐
for user in users {
   if user.id == 1 { return true }
}
```

#### force_cast

不建议直接强解类型

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| force_cast | 启用 | no | idiomatic |

示例

```
// 推荐
NSNumber() as? Int

// 不推荐
NSNumber() as! Int
```

#### force_try

对会抛出异常(throws)的方法,不建议try!强解

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| force_try | 启用 | no | idiomatic |

示例

```
func myFunction() throws { }

// 推荐
do {
    try myFunction()
} catch {

}

// 不推荐，这样直接触发 error
try! myFunction()
```

#### force_unwrapping

强制解包/拆包。我们知道，当一个类型是可选类型的时候，当我们获取值时，需要强制解包（也叫隐式解包）, 通常我们是在一个变量或者所需要的常量、类型等后面加一个“ ！”， 然而，swiftlint建议强制解包应该要避免， 否则将给予warning

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| force_unwrapping | 未启用 | no | idiomatic |

示例

```
// 推荐，不会触发warning
navigationController?.pushViewController(myViewController, animated: true)

/// 不推荐，将触发warning
navigationController!.pushViewController(myViewController, animated: true)

let url = NSURL(string: "http://www.baidu.com")!
print(url)

return cell!
```

#### function_body_length

函数体长度， 函数体不应该跨越太多行， 超过40行给warning， 超过100行直接报错，*可以禁用规避掉*

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| function_body_length | 启用 | no | metrics |

#### function_parameter_count

函数参数个数 函数参数数量(init方法除外)应该少点，不要太多，swiftlint规定函数参数数量超过5个给warning， 超过8个直接报error，*可以禁用规避掉*

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| function_parameter_count | 启用 | no | metrics |

#### generic_type_name

泛型类型名称只能包含字母数字字符，以大写字母开头，长度介于1到20个字符之间

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| generic_type_name | 未启用 | no | idiomatic |

示例

```
// 推荐
func foo<T>() {}
func foo<T>() -> T {}
func foo<T, U>(param: U) -> T {}
func foo<T: Hashable, U: Rule>(param: U) -> T {}

// 不推荐
func foo<T_Foo>() {}
func foo<T, U_Foo>(param: U_Foo) -> T {}
func foo<TTTTTTTTTTTTTTTTTTTTT>() {}
func foo<type>() {}
```

#### identifier_name

变量标识符名称应该只包含字母数字字符，并以小写字母开头或只应包含大写字母。在上述例外情况下，当变量名称被声明为静态且不可变时，变量名称可能以大写字母开头。变量名称不应该太长或太短

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| identifier_name | 启用 | no | style |

示例

```
internal struct IdentifierNameRuleExamples {
    // 不会触发error
    static let nonTriggeringExamples = [
        "let myLet = 0",
        "var myVar = 0",
        "private let _myLet = 0",
        "class Abc { static let MyLet = 0 }",
        "let URL: NSURL? = nil",
        "let XMLString: String? = nil",
        "override var i = 0",
        "enum Foo { case myEnum }",
        "func isOperator(name: String) -> Bool",
        "func typeForKind(_ kind: SwiftDeclarationKind) -> String",
        "func == (lhs: SyntaxToken, rhs: SyntaxToken) -> Bool",
        "override func IsOperator(name: String) -> Bool"
    ]

    // 会触发error
    static let triggeringExamples = [
        "↓let MyLet = 0",
        "↓let _myLet = 0",
        "private ↓let myLet_ = 0",
        "↓let myExtremelyVeryVeryVeryVeryVeryVeryLongLet = 0",
        "↓var myExtremelyVeryVeryVeryVeryVeryVeryLongVar = 0",
        "private ↓let _myExtremelyVeryVeryVeryVeryVeryVeryLongLet = 0",
        "↓let i = 0",
        "↓var id = 0",
        "private ↓let _i = 0",
        "↓func IsOperator(name: String) -> Bool",
        "enum Foo { case ↓MyEnum }"
    ]
}
```

#### implicit_getter

对于只有只读属性不建议重写get方法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| implicit_getter | 启用 | no | style |

示例

```
//不会触发error
//重写get和set方法
class Foo {
  var foo: Int {
    get {
      return 3
    }
    set {
      _abc = newValue
    }
  }
}
//只读
class Foo {
  var foo: Int {
     return 20
  }
}

class Foo {
  static var foo: Int {
     return 20
  }
}


//会触发error
class Foo {
  var foo: Int {
    get {
      return 20
    }
  }
}

class Foo {
  var foo: Int {
    get{
      return 20
    }
  }
}
```

#### implicit_return

建议使用隐式返回闭包; 如: foo.map({ $0 + 1 })

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| implicit_return | 未启用 | no | style |

示例

```
// 推荐
foo.map { $0 + 1 }
foo.map({ $0 + 1 })
foo.map { value in value + 1 }

// 不推荐
foo.map { value in
  return value + 1
}
foo.map {
  return $0 + 1
}
```

#### implicitly_unwrapped_optional

尽量避免隐式解析可选类型的使用

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| implicitly_unwrapped_optional | 未启用 | no | idiomatic |

示例

```
public static let description = RuleDescription(
    identifier: "implicitly_unwrapped_optional",
    name: "Implicitly Unwrapped Optional",
    description: "Implicitly unwrapped optionals should be avoided when possible.",
    kind: .idiomatic,

    //不会触发warning
    nonTriggeringExamples: [
        "@IBOutlet private var label: UILabel!",
        "@IBOutlet var label: UILabel!",
        "@IBOutlet var label: [UILabel!]",
        "if !boolean {}",
        "let int: Int? = 42",
        "let int: Int? = nil"
    ],

    //会触发warning
    triggeringExamples: [
        "let label: UILabel!",
        "let IBOutlet: UILabel!",
        "let labels: [UILabel!]",
        "var ints: [Int!] = [42, nil, 42]",
        "let label: IBOutlet!",
        "let int: Int! = 42",
        "let int: Int! = nil",
        "var int: Int! = 42",
        "let int: ImplicitlyUnwrappedOptional<Int>",
        "let collection: AnyCollection<Int!>",
        "func foo(int: Int!) {}"
    ]
)
```

#### is_disjoint

初始化集合Set时,推荐使用Set.isDisjoint(), 不建议:Set.intersection

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| is_disjoint | 启用 | no | idiomatic |

示例

```
// 推荐写法
_ = Set(syntaxKinds).isDisjoint(with: commentAndStringKindsSet)
let isObjc = !objcAttributes.isDisjoint(with: dictionary.enclosedSwiftAttributes)
```

#### joined_default_parameter

joined方法使用默认分隔符时, 建议使用joined()方法, 而不是joined(separator: "")方法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| joined_default_parameter | 未启用 | yes | idiomatic |

示例

```
// 推荐
let foo = bar.joined()
let foo = bar.joined(separator: ",")
let foo = bar.joined(separator: toto)

// 不推荐
let foo = bar.joined(separator: "")
let foo = bar.filter(toto).joined(separator: "")
```

#### large_tuple

定义的元组成员个数,超过两个warning

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| large_tuple | 启用 | no | metrics |

示例

```
// 不推荐
let foo: (Int, Int)
let foo: (start: Int, end: Int)
let foo: (Int, (Int, String))

// 推荐
let foo: (Int, Int, Int)
let foo: (start: Int, end: Int, value: String)
let foo: (Int, (Int, Int, Int))
```

#### leading_whitespace

文件开始不应该有空格或者换行, 否则就会触发warning

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| leading_whitespace | 启用 | yes | style |

#### legacy_cggeometry_functions

当获取某个视图的宽、高、最小X、最大X值等等，swiftlint推荐使用swift的标准语法，尽量不要使用从Objective-C中的遗留版本，尽量语法swift化

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| legacy_cggeometry_functions | 启用 | yes | idiomatic |

示例

```
// 推荐
rect.width
rect.height
rect.minX
rect.midX
rect...................

// 不推荐
CGRectGetWidth(someView.frame)
```

#### legacy_constant

和属性 `legacy_cggeometry_functions` 一样， 结构范围常数尽量分开、明确、具体， 不要使用OC的遗留整体常数

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| legacy_constant | 启用 | yes | idiomatic |

示例

```
// 推荐
CGPoint.zero

// 不推荐
CGPointZero
CGRectZero
```

#### legacy_constructor

swiftlint要求系统自带构造器， 使用swift语法化，不要使用OC版本的构造器

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| legacy_constructor | 启用 | yes | idiomatic |

示例

```
// swift语法，相信之后系统也会强制规定使用
CGPoint（x: 10， y: 20）

// 错误的构造器语法
CGPointMake(10, 20)
```

#### legacy_nsgeometry_functions

ns类几何函数， 和前面的几个属性一样， 使用swift点语法函数， 不使用以前的版本。

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| legacy_nsgeometry_functions | 启用 | yes | idiomatic |

示例

```
/// 正确
view.width/height/minX

/// 错误
NSWidth(view.frame)
```

#### let_var_whitespace

let和var语句应该用空白行与其他语句分开

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| let_var_whitespace | 未启用 | no | style |

示例

```
// 推荐
let a = 0
var x = 1

x = 2

// 不推荐
var x = 1
x = 2
```

#### line_length

行的字符长度属性。官方的规定是超过120字符就给warning， 超过200个字符就直接报error！

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| line_length | 启用 | no | metrics |

#### literal_expression_end_indentation

字典和数组的开头和结尾要有相同的缩进格式

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| literal_expression_end_indentation | 未启用 | no | style |

#### mark

标记方法或者属性。

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| mark | 启用 | yes | lint |

示例

```
// 推荐
// MARK: good
// MARK: - good
// MARK: -

// 不推荐
//MARK: bad
// MARK:bad
//MARK:bad
//  MARK: bad
// MARK:  bad
// MARK: -bad
```

#### multiline_arguments

调用函数和方法时, 其参数应该在同一行上，或者每行一个

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| multiline_arguments | 未启用 | no | style |

示例

```
// 不推荐
foo(0, param1: 1, param2: true,
    param3: [3])

foo(
    0, param1: 1,
    param2: true, param3: [3]
)
```

#### multiline_parameters

声明函数和方法时, 其参数应该在同一行上，或者每行一个

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| multiline_parameters | 未启用 | no | style |

示例

```
// 不推荐
protocol Foo {
   func foo(param1: Int,
             param2: Bool, param3: [String]) { }
}

protocol Foo {
   func foo(param1: Int, param2: Bool,
             param3: [String]) { }
}
```

#### multiple_closures_with_trailing_closure

当函数有多个闭包时, 不建议使用尾随闭包语法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| multiple_closures_with_trailing_closure | 启用 | no | style |

示例

```
// 不推荐
foo.something(param1: { $0 }) { $0 + 1 }

UIView.animate(withDuration: 1.0, animations: {
    someView.alpha = 0.0
}) { _ in
    someView.removeFromSuperview()
}
```

#### nesting

嵌套。类型嵌套至多一级结构， 函数语句嵌套至多五级结构。

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| nesting | 启用 | no | metrics |

#### nimble_operator

快捷操作符。和自由匹配函数相比，更喜欢快捷操作符，比如：>=、 ==、 <=、 <等等。

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| nimble_operator | 未启用 | yes | idiomatic |

示例

```
// 推荐
(person.voice) != "Hello world"   // 判断字符串相同
10 > 5                // 10比5大
99 < 100              // 99比100小

// 不推荐
(person.voice).toNot(equal("Hello world"))     // 判断字符串相同
10.to(beGreaterThan(5))     // 10比5大
99.to(beLessThan(100))       // 99比100小
```

#### no_grouping_extension

只有class和protocol可以使用extension,其他类型不可使用

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| no_grouping_extension | 未启用 | no | idiomatic |

示例

```
// 不推荐
enum Fruit {}
extension Fruit {}

extension Tea: Error {}
struct Tea {}

class Ham { class Spam {}}
extension Ham.Spam {}

extension External { struct Gotcha {}}
extension External.Gotcha {}
```

#### notification_center_detachment

对象移除通知只能在deinit移除self,函数中不能removeObserver(self)

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| notification_center_detachment | 启用 | no | lint |

示例

```
// 不会触发warning
class Foo {
   deinit {
       NotificationCenter.default.removeObserver(self)
   }
}

class Foo {
   func bar() {
       NotificationCenter.default.removeObserver(otherObject)
   }
}


// 会触发warning
class Foo {
   func bar() {
       NotificationCenter.default.removeObserver(self)
   }
}
```

#### number_separator

数字分割线。当在大量的小数中， 应该使用下划线来作为千分位分割线

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| number_separator | 未启用 | yes | style |

示例

```
// 推荐
let xxx = 1_000_000_000.000_1
print(xxx)

// 不推荐
let xxx = 1000000000.0001
print(xxx)
```

#### object_literal

swiftlint表示比起图片和颜色初始化，更喜欢对象初始化。因为swift初始化可以用表情，图片，颜色等，这不符合项目中的一些习惯用法。
与 `discouraged_object_literal` 互斥， *个人不推荐*

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| object_literal | 未启用 | no | idiomatic |

示例

```
// 推荐
let image = #imageLiteral(resourceName: "image.jpg")
let color = #colorLiteral(red: 0.9607843161, green: 0.7058823705, blue: 0.200000003, alpha: 1)
let image = UIImage(named: aVariable)
let image = UIImage(named: "interpolated \(variable)")
let color = UIColor(red: value, green: value, blue: value, alpha: 1)
let image = NSImage(named: aVariable)
let image = NSImage(named: "interpolated \(variable)")
let color = NSColor(red: value, green: value, blue: value, alpha: 1)

// 不推荐
let image = ↓UIImage(named: "foo")
let color = ↓UIColor(red: 0.3, green: 0.3, blue: 0.3, alpha: 1)
let color = ↓UIColor(red: 100 / 255.0, green: 50 / 255.0, blue: 0, alpha: 1)
let color = ↓UIColor(white: 0.5, alpha: 1)
```

#### opening_brace

花括号之前应该有一个空格,且与声明在同一行

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| opening_brace | 启用 | yes | style |

示例

```
// 建议
func abc() {
}
[].map() { $0 }
[].map({ })
if let a = b { }
while a == b { }
guard let a = b else { }
```

#### operator_usage_whitespace

操作符使用规则，操作符两边应该有空格。比如 “+” “-” “？？”

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| operator_usage_whitespace | 未启用 | yes | style |

示例

```
// 推荐
let foo = 1 + 2
let foo = 1 > 2
let foo = !false

// 不推荐
let foo = 1+2
let foo = 1   + 2
let foo = 1   +    2
let foo = 1 +    2
let foo=1+2
```

#### operator_whitespace

空格/空白操作符。当定义空格操作符的时候，被定义的名字或类型两边应该各有一个单行空格操作符

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| operator_whitespace | 启用 | no | style |

示例

```
// 触发警告
class Something: Equatable {
    var text: String?
    // "=="和“(lhs: Something, rhs: Something)”之间应该有一个空格
    static func ==(lhs: Something, rhs: Something) -> Bool {
        return lhs.text == rhs.text
    }
}
```

#### overridden_super_call

一些重写的方法应该调用super.(父类的)方法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| overridden_super_call | 未启用 | no | lint |

#### override_in_extension

在extension中,不能重写未声明的属性和未定义的方法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| override_in_extension | 未启用 | no | lint |

示例

```
// 错误写法
extension Person {
  //该属性之前未定义, 不能重写
  override var age: Int { return 42 }
}

extension Person {
  //该方法之前也为定义不能重写
  override func celebrateBirthday() {}
}
```

#### pattern_matching_keywords

在switch-case语句中, 建议不要将case中的let和var等关键字放到元祖内

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| pattern_matching_keywords | 未启用 | no | style |


#### prefixed_toplevel_constant

类似全局常量,建议前缀以k开头

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| prefixed_toplevel_constant | 未启用 | no | style |

示例

```
// 推荐
private let kFoo = 20.0
public let kFoo = false
internal let kFoo = "Foo"
let kFoo = true
```

#### private_action

IBActions修饰的方法,应该都是私有的

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| private_action | 未启用 | no | lint |

#### private_outlet

IBOutlets修饰的属性应该都是私有的

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| private_outlet | 未启用 | no | lint |

#### private_over_fileprivate

private比fileprivate的私有程度更高

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| private_over_fileprivate | 启用 | yes | idiomatic |

#### private_unit_test

私有的单元测试。被标记为private的单元测试不会被测试工具XCTest运行， 也就是说，被标记为private的单元测试会被静态跳过

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| private_unit_test | 启用 | no | lint |

示例

```
private ↓class FooTest: XCTestCase { ...............继承于测试用例类XCTestCase, 被标记为private，所以触发warning
     func test1() {}
     internal func test2() {}
     public func test3() {}
     private func test4() {}.......另外注意这里，上面既然不会通过，那显然这里也不会通过，根本不会走这个func
 }

 internal class FooTest: XCTestCase { ......开始通过测试，因为没有被标记为private
     func test1() {}
     internal func test2() {}
     public func test3() {}
     private ↓func test4() {}................不通过，因为被标记为private
 }

 public class FooTest: XCTestCase { ..........通过
     func test1() {}
     internal func test2() {}
     public func test3() {}
     private ↓func test4() {}.................不通过，因为被标记成private
 }

 class FooTest: XCTestCase { ..........通过
     func test1() {}
     internal func test2() {}
     public func test3() {}
     private ↓func test4() {}.................不通过，因为被标记成private
 }
```

#### prohibited_super_call

一些方法不应该调用父类的方法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| prohibited_super_call | 未启用 | no | lint |

示例

```
//以下方法不建议调用父类的方法
class VC: UIViewController {
    override func loadView() {↓
        super.loadView()
    }
}
class VC: NSFileProviderExtension {
    override func providePlaceholder(at url: URL,completionHandler: @escaping (Error?) -> Void) {↓
        self.method1()
        super.providePlaceholder(at:url, completionHandler: completionHandler)
    }
}
class VC: NSView {
    override func updateLayer() {↓
        self.method1()
        super.updateLayer()
        self.method2()
    }
}
class VC: NSView {
    override func updateLayer() {↓
        defer {
            super.updateLayer()
        }
    }
}
```

#### protocol_property_accessors_order

在协议中声明属性时，访问者的顺序应该是get set

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| protocol_property_accessors_order | 启用 | yes | style |

示例

```
// 建议
protocol Foo {
   var bar: String { get set }
}

// 不建议
protocol Foo {
   var bar: String { set get }
}
```

#### quick_discouraged_call

在单元测试中,不建议在describe和content比保重直接调用方法和类

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| quick_discouraged_call | 未启用 | no | lint |

#### quick_discouraged_focused_test

在单元测试中,不建议集中测试,否则可能不能运行成功

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| quick_discouraged_focused_test | 未启用 | no | lint |

#### quick_discouraged_pending_test

单元测试中阻止未进行的测试单元

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| quick_discouraged_pending_test | 未启用 | no | lint |

#### redundant_discardable_let

不需要初始化方法返回结果时,建议使用: _ = Person(), 而不是:let _ = Person()

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| redundant_discardable_let | 启用 | yes | style |

示例

```
// 推荐
_ = foo()
if let _ = foo() { }
guard let _ = foo() else { return }

// 不建议
let _ = foo()
if _ = foo() { let _ = bar() }
```

#### redundant_nil_coalescing

使用可能为为nil的可选值时,建议使用: str ?? "", ??左右两侧要有一个空格

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| redundant_nil_coalescing | 未启用 | yes | idiomatic |

示例

```
// 建议
var myVar: Int?; myVar ?? 0

// 不建议
var myVar: Int? = nil; myVar  ?? nil
var myVar: Int? = nil; myVar??nil
```

#### redundant_optional_initialization

初始化nil变量时,不建议赋值nil

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| redundant_optional_initialization | 启用 | yes | idiomatic |

示例

```
// 不会触发warning
var myVar: Int?
let myVar: Int? = nil
var myVar: Optional<Int>
let myVar: Optional<Int> = nil

// 会触发warning
var myVar: Int?↓ = nil
var myVar: Optional<Int>↓ = nil
var myVar: Int?↓=nil
var myVar: Optional<Int>↓=nil
```

#### redundant_string_enum_value

在定义字符串枚举的时候, 当字符串枚举值等于枚举名称时，可以不用赋值

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| redundant_string_enum_value | 启用 | no | idiomatic |

示例

```
// 不会触发warning
enum Numbers: String {
 case one
 case two
}
enum Numbers: Int {
 case one = 1
 case two = 2
}

// 会触发warning
enum Numbers: String {
 case one = ↓"one"
 case two = ↓"two"
}
enum Numbers: String {
 case one = ↓"one", two = ↓"two"
}
```

#### redundant_void_return

当函数返回值为Void时,建议不写返回值, 定义常量或者变量的时候可以

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| redundant_void_return | 启用 | yes | idiomatic |

示例

```
// 不会触发warning
func foo() {}
func foo() -> Int {}
func foo() -> Int -> Void {}
func foo() -> VoidResponse
let foo: Int -> Void

// 会触发warning
func foo()↓ -> Void {}
protocol Foo {
 func foo()↓ -> Void
}
func foo()↓ -> () {}
protocol Foo {
 func foo()↓ -> ()
}
```

#### required_enum_case

定义的枚举,必须有与其对应的操作实现

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| required_enum_case | 未启用 | no | lint |

#### return_arrow_whitespace

swiftlint推荐返回箭头和返回类型应该被空格分开

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| return_arrow_whitespace | 启用 | yes | style |

示例

```
// 推荐写法
func abc() -> Int {}
func abc() -> [Int] {}

// 不建议写法
func abc()↓->Int {}
func abc()↓->[Int] {}
func abc()↓->(Int, Int) {}
func abc()↓-> Int {}
func abc()↓ ->Int {}
```

#### shorthand_operator

在swiftlint中， 就是我们常用的简洁操作运算符，比如：+= ， -=， *=， /= 等等。在swiftlint中，在做一些赋值操作的时候，推荐使用简短操作符

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| shorthand_operator | 启用 | no | style |

示例

```
/// 不推荐使用
var value = 4
value = value / 2
print(value)

/// 推荐使用
var value = 4
value /= 2
print(value)
```

#### single_test_class

单元测试中,测试文件应该包含一个QuickSpec或XCTestCase类

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| single_test_class | 未启用 | no | style |

#### sorted_first_last

在获取某数组中最大最小值时,建议使用min和max函数,而不是sorted().first和sorted().lase

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| sorted_first_last | 未启用 | no | style |

#### sorted_imports

分类/有序导入。 这个属性有些奇怪， 要求导入的时候导入的类要按顺序导入

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| sorted_imports | 未启用 | yes | style |

示例

```
//建议写法
import AAA
import BBB
import CCC
import DDD


import Alamofire
import API
```

#### statement_position

陈述句位置， 这里主要指的是 else 和 catch 前面要加一个空格， 也不能大于1个空格， 否则就会触发警告

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| statement_position | 启用 | yes | style |

示例

```
/// 没有空格，触发warning
let number = "long"
if number.isEmpty {
    print("为空")
}else {.............................注意这里
    print("不为空")
}

/// 这里也会触发warning， 因为else if换行了
let number = "long"
if number.isEmpty {
    print("为空")
}
else if number.contains("long") {............................注意这里
    print("不为空")
} else {
    print("s")
}

/// 正确的写法
let number = "long"
if number.isEmpty {
    print("为空")
} else {
    print("不为空")
}
```

#### strict_fileprivate

extension中不建议使用fileprivate 修饰方法和属性

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| strict_fileprivate | 未启用 | no | idiomatic |

#### superfluous_disable_command

被禁用的规则不会在禁用区域触发警告

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| superfluous_disable_command | 启用 | no | lint |

#### switch_case_alignment

switch-case语句中switch和case应该垂直对齐

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| switch_case_alignment | 启用 | no | style |

示例

```
// 推荐
switch someBool {
case true: // case 1
    print('red')
case false:
    if case let .someEnum(val) = someFunc() {
        print('blue')
    }
}

// 不推荐
switch someBool {
    ↓case true:
         print('red')
    ↓case false:
         print('blue')
}
```

#### switch_case_on_newline

在switch语法里， case应该总是在一个新行上面

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| switch_case_on_newline | 启用 | no | idiomatic |

示例

```
// swiftlint表示会触发warning
switch type {
case .value1: print("1")...................在同一行错
case .value2: print("2")...................在同一行错
default: print("3")...................在同一行错
}

/// 不会触发warning
switch type {
case .value1:
    print("1")
case .value2:
    print("2")
default:
    print("3")
}
```

#### syntactic_sugar

swiftlint推荐使用速记语法糖， 例如 [Int] 代替 Array, 强烈建议推荐使用

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| syntactic_sugar | 启用 | no | idiomatic |

示例

```
/// 触发warning
let myArray: Array<Int> = [1, 2, 3]
print(myArray)

/// 正确写法，不会触发warning
let myArray: [Int] = [1, 2, 3]
    print(myArray)
```

#### todo

TODO 和 FIXME 应该避免使用， 使用“notaTODO 和 notaFIXME”代替。另外， 和 MARK 标记不同的是， “notaTODO 和 notaFIXME”没有空格要求

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| todo | 启用 | no | lint |

#### trailing_closure

关于闭包中{}的使用, 推荐使用尾随闭包的语法

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| trailing_closure | 未启用 | no | style |

示例

```
//推荐使用
foo.map { $0 + 1 }
foo.reduce(0) { $0 + 1 }

//不推荐使用
foo.map({ $0 + 1 })
↓foo.reduce(0, combine: { $0 + 1 })
```

#### trailing_comma

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| trailing_comma | 启用 | yes | style |

#### trailing_newline

文件（属性、方法）结束的的时候（“}”之前）， 应该有一个空格新行，但这里要注意的是

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| trailing_newline | 启用 | yes | style |

示例

```
/// 空一行，不会触发警告
nonTriggeringExamples: [
    "let a = 0\n"
],

/// 下面会触发警告
triggeringExamples: [
    "let a = 0",   /// 不空行，会触发警告（实际上，我试过，不会触发警告）
    "let a = 0\n\n"   /// 空两行， 会触发警告（实际上，我试过，会触发警告，但是触发的是vertical_whitespace警告而不是trailing_newline）
],

/// 说说这里，它要求改正为都空一行，虽然这样code看起来很轻松，但如果定义变量或常量太多，就太分散了（值得说的是，就算不空行也不会触发trailing_newline, 应该刚才也已经说了，这个属性只是说“应该”，而不是必须）
corrections: [
    "let a = 0": "let a = 0\n",
    "let b = 0\n\n": "let b = 0\n",
    "let c = 0\n\n\n\n": "let c = 0\n"
]
```

#### trailing_semicolon

尽管在变量或常量赋值之后加不加分号在swift中没有硬性的要求，但是为了使code style更swift化，所以尽量或者绝对不要加“;”

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| trailing_semicolon | 启用 | yes | idiomatic |

#### trailing_whitespace

函数方法结束后,不建议添加空格行, 和vertical_whitespace貌似有冲突

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| trailing_whitespace | 启用 | yes | style |

示例

```
/// 下面这个例子不会触发警告，但是一旦其中有一个空行就会触发警告trailing_whitespace, 这和vertical_whitespace实质上有些冲突，vertical_whitespace要求两行code之间不超过1行，要么没有空行，要么只有1行，而trailing_whitespace要求没有空行！！！
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let a = 0
        let b = 1
        let c = 2
    }
    func chenlong() -> Void {
        let a = 0
        print(a)
    }
}
```

#### type_body_length

类型体长度。类型体长度不应该跨越太多行， 超过200行给warning，超过350行给error。一般是大括号或者括号内, 比如定义一个enum或struct

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| type_body_length | 启用 | no | metrics |

#### type_name

类型名， 类型名应该只包含字母数字字符， 并且以大写字母开头，长度在3-40个字符

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| type_name | 启用 | no | idiomatic |

#### unneeded_break_in_switch

在switch-case语句中, 有方法调用或操作时,避免使用break语句

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| unneeded_break_in_switch | 启用 | no | idiomatic |

示例

```
//不会触发warning
switch foo {
case .bar:
    break
}
switch foo {
default:
    break
}
switch foo {
case .bar:
    something()
}

//会触发warning
switch foo {
case .bar:
//这里已经有方法调用了
    something()
    ↓break
}
```

#### unneeded_parentheses_in_closure_argument

在定义或使用闭包时,闭包参数不建议使用括号()

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| unneeded_parentheses_in_closure_argument | 未启用 | yes | style |

示例

```
//建议
let foo = { (bar: Int) in }
let foo = { bar, _  in }
let foo = { bar in }

//不建议
call(arg: { ↓(bar) in })
call(arg: { ↓(bar, _) in })
```

#### unused_closure_parameter

swiftlint建议最好把不使用的闭包参数使用 “_”代替

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| unused_closure_parameter | 启用 | yes | lint |

示例

```
//不会触发warning
[1, 2].map { number in
 number + 1
}
[1, 2].map { _ in
 3
}

//会触发warning
[1, 2].map { ↓number in
 return 3
}
[1, 2].map { ↓number in
 return numberWithSuffix
}
```

#### unused_enumerated

在for遍历数组时, 如有未使用的索引,不建议使用.enumerated()

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| unused_enumerated | 启用 | no | idiomatic |

示例

```
//不会触发warning
for (idx, foo) in bar.enumerated() { }
for (_, foo) in bar.enumerated().something() { }
for (_, foo) in bar.something() { }

//会触发warning
for (↓_, foo) in bar.enumerated() { }
for (↓_, foo) in abc.bar.enumerated() { }
for (↓_, foo) in abc.something().enumerated() { }
for (idx, ↓_) in bar.enumerated() { }
```

#### unused_optional_binding

在使用if判断某变量是否为nil的时候, 不建议使用下划线(_)

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| unused_optional_binding | 启用 | no | style |

示例

```
//不会触发warning
if let bar = Foo.optionalValue {
}

//会触发warning
if let ↓_ = Foo.optionalValue {
}
if let a = Foo.optionalValue, let ↓_ = Foo.optionalValue2 {
}
```

#### valid_ibinspectable

@IBInspectable在swiftlint中的使用需要注意， 第一必须是变量， 第二必须要有指定的类型，如果指定的类型是可选类型或者隐式类型，则目前官方只支持以下几种类型：String, NSString, UIColor, NSColor, UIImage, NSImage.

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| valid_ibinspectable | 启用 | no | lint |

示例

```
/// 指定为变量var， 类型为String？和String！
@IBInspectable private var yyy: String?
@IBInspectable private var zzz: String!

/// 如果写成这样，编译能通过，但是会触发警告, 因为swiftlint暂不支持Int可选和隐式类型:
@IBInspectable private var dddl: Int!
@IBInspectable private var eeel: Int?

/// 如果指定的类型不是可选类型， 就应该初始化，否则系统不允许，会报错所在的类没有初始化
对：
@IBInspectable private var counts: Int = 0
系统报错：
@IBInspectable private var counts: Int
```

#### vertical_parameter_alignment

垂直方向上的参数对齐。当函数参数有多行的时候， 函数参数在垂直方向上应该对齐（参数换行的时候左边对齐）

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| vertical_parameter_alignment | 启用 | no | style |

示例

```
//不会触发warning
func validateFunction(_ file: File, kind: SwiftDeclarationKind,
                      dictionary: [String: SourceKitRepresentable]) { }

func validateFunction(_ file: File, kind: SwiftDeclarationKind,
                      dictionary: [String: SourceKitRepresentable])
                      -> [StyleViolation]
//会触发warning
func validateFunction(_ file: File, kind: SwiftDeclarationKind,
                  ↓dictionary: [String: SourceKitRepresentable]) { }
```

#### vertical_parameter_alignment_on_call

当调用多个参数的函数时,如果参数多行显示,则应该垂直对齐

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| vertical_parameter_alignment_on_call | 未启用 | no | style |

示例

```
//不会触发warning
foo(param1: 1, param2: bar
    param3: false, param4: true)
foo(param1: 1, param2: bar)
foo(param1: 1, param2: bar
    param3: false,
    param4: true)


//会触发warning
foo(param1: 1, param2: bar
                ↓param3: false, param4: true)

foo(param1: 1, param2: bar
 ↓param3: false, param4: true)

foo(param1: 1, param2: bar
       ↓param3: false,
       ↓param4: true)
```

#### vertical_whitespace

垂直方向上的空格行，限制为一行（注释除外）

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| vertical_whitespace | 启用 | yes | style |

示例

```
/// 没有空格， nonTriggerWarning
override func viewDidLoad() {
    super.viewDidLoad()
    let aaa = 0
}

/// 有一行空格, nonTriggerWarning
override func viewDidLoad() {
    super.viewDidLoad()
    let aaa = 0
    ............................1
}

/// >=2行，就会触发警告
override func viewDidLoad() {
    super.viewDidLoad()
    let aaa = 0
   .............................1
   .............................2
}
```

#### void_return

多余的返回值为空， 在函数声明的时候，返回值为空是多余的。定义常量或者变量的时候可以

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| void_return | 启用 | yes | style |

示例

```
/// 这个属性要求这样写， 返回值为空省略
func XingYun() {
    print("titan")
}

/// 这个属性要求别这样写，否则会有warning（但是我在swift 3.0上测试并没有触发warning）
func XingYun() -> Void {
    print("titan")
}
```

#### weak_delegate

代理应该写成weak类型（弱代理）来避免循环引用

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| weak_delegate | 启用 | no | lint |

示例

```
/// 1.1 编译通过
class Langke {
    var chenlong: NSObjectProtocol?
}

/// 1.2 编译通过，但是触发swiftlint的 weak_delegate警告， 原因是变量名 myDelegate 中有 delegate 关键字，这属于名字滥用
class Langke {
    var myDelegate: NSObjectProtocol?
}

/// 1.3 编译通过， 不会触发警告， 原因是在 var 关键字前面加了 weak
class Langke {
    weak var myDelegate: NSObjectProtocol?
}


/// 2.1 编译通过，但是触发 weak_delegate 警告，原因是 scrollDelegate 中 Delegate 放在了最后， 被理解成了代理
class Langke {
    var scrollDelegate: UIScrollViewDelegate?
}

/// 2.2 编译通过， 既然变量名被理解成了代理， 那为了类似防止循环引用， 应该加关键字 weak
class Langke {
    weak var scrollDelegate: UIScrollViewDelegate?
}

/// 编译通过， 不会触发警告， 因为delegate放在了前面， 没有被理解成代理
class Langke {
    var delegateScroll: UIScrollViewDelegate?
}
```

#### xctfail_message

单元测试中,XCTFail调用应该包括声明描述

| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| xctfail_message | 启用 | no | idiomatic |

#### yoda_condition


| 识别码 | 默认是否启用 | 是否支持自动更正 | 类型 | 默认配置 |
| --- | --- | --- | --- | --- |
| yoda_condition | 未启用 | no | lint |

示例

```
//不会触发warning
if foo == 42 {}
if foo <= 42.42 {}
guard foo >= 42 else { return }
guard foo != "str str" else { return }

//会触发warning
↓if 42 == foo {}
↓if 42.42 >= foo {}
↓guard 42 <= foo else { return }
↓guard "str str" != foo else { return }
↓while 10 > foo { }
```