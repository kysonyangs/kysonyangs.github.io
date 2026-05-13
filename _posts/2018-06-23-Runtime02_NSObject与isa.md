---
layout: post
title: "Runtime02 NSObject与isa"
date: 2018-06-23 16:04:10 +0800
tags: ["Runtime"]
---

## NSObject底层

看下NSObject中的定义:

```
@interface NSObject <NSObject>
    Class isa;
@end
```

其实NSObject就是一个Class对象，不过是对变量isa封装了一系列的操作而已，那么Class又是什么类型呢？在objc-runtime-new.h可以找到其定义，是指向结构体objc_class的指针，如下：
<!-- more -->
```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY; //isa指针
} OBJC2_UNAVAILABLE;

struct objc_class : objc_object {
    // Class ISA;
    Class superclass; // superclass指针
    cache_t cache;             // 方法缓存
    class_data_bits_t bits;    // 数据

    // 通过(bits & FAST_DATA_MASK)获取 class_rw_t 数据
    class_rw_t *data() {
        return bits.data();
    }
    ...
}

struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro; //只读属性

    method_array_t methods; // 方法列表二维数组
    property_array_t properties; //属性列表
    protocol_array_t protocols; //协议列表
    ...
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize; //instance对象所占内存大小
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;

    const char * name; // 类名
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars; // 成员变量列表

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```

再来看一张图

![](https://s1.ax1x.com/2020/05/13/YwSu9S.png)

## 实例、类、元类 存储信息

instance对象在内存中存储的信息包括
- isa 指针
- 其他成员变量

class类对象在内存中存储的信息主要包括
- isa 指针
- superclass 指针
- 类的属性信息（@property）
- 类的对象方向方法信息（instance method）
- 类的协议信息（protocol）
- 类的成员变量信息（ivar）
- ...

meta-class元类对象和class对象的内存结构是一样的，但是用途不一样，在内存中存储的信息主要包括
- isa 指针
- superclass 指针
- 类的类方向方法信息（class method）
- ....

## isa指针

在arm64架构之前，isa就是一个普通的指针，存储着Class、Meta-Class对象的内存地址

从arm64架构开始，对isa进行了优化，变成了一个共用体（union）结构，还使用位域来存储更多的信息

```
union isa_t
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

    // arm64架构下
    uintptr_t nonpointer        : 1;   // 代表isa的类型，如果为0，则isa中只保存着class的信息，则是之前32位机器时的设计raw isa，如果为1，class信息保存在shiftcls中，其它空间用于保存额外的信息。
    uintptr_t has_assoc         : 1;   // 对象含有或者曾经含有关联引用，没有关联引用的可以更快地释放内存
    uintptr_t has_cxx_dtor      : 1;   // 这一位表示当前对象有 C++ 或者 ObjC 的析构器(destructor)，如果没有析构器就会快速释放内存
    uintptr_t shiftcls          : 44;  // 保存对应的Class指针信息，可以看到复制代码是这样的`newisa.shiftcls = (uintptr_t)cls >> 3`，右移3位的目的是用于将 Class 指针中无用的后三位清除减小内存的消耗，因为类的指针要按照字节（8 bits）对齐内存，其指针后三位都是没有意义的 0。
    uintptr_t magic             : 6;   // 初始化之后值为0x3d，用于调试器判断当前对象是真的对象还是没有初始化的空间
    uintptr_t weakly_referenced : 1;   // 对象被指向或者曾经指向一个 ARC 的弱变量，没有弱引用的对象可以更快释放
    uintptr_t deallocating      : 1;   // 对象正在释放内存
    uintptr_t has_sidetable_rc  : 1;   // 对象的引用计数太大了，存不下，要用辅助的sidetable来保存
    uintptr_t extra_rc          : 8;   // 对象的引用计数超过 1，会存在这个这个里面，如果引用计数为 6，extra_rc 的值就为 5
....
};
```