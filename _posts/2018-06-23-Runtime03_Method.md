---
layout: post
title: "Runtime03 Method"
date: 2018-06-23 18:30:50 +0800
tags: ["Runtime"]
---

- class_rw_t里面的methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容、分类的内容

![](https://s1.ax1x.com/2020/05/13/YwCE7R.png)

- class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的，包含了类的初始内容

![](https://s1.ax1x.com/2020/05/13/YwCZA1.png)
<!-- more -->
然后看下方法的结构
```
struct method_t{
  SEL name; // 方法名
  const char *types; // 参数和返回类型encode
  IMP imp; // 函数指针
};
typedef struct method_t *Method;
```

看下class_rw_t methods 的初始化, class_ro_t 的结构在编译期间已经决定，无法更改
```
static Class realizeClass(Class cls){
  const class_ro_t *ro = (const class_ro_t *)cls->data();
  class_rw_t *rw;

  if(ro->flags & RO_FUTERE){
    // rw已经分配好了
    rw = cls->data();
    ro = cls->data()->ro;
    cls->changeInfo(RW_REALIZED|RW_REALIZING,RW_FUTURE);
  }else{
    // 未初始化
    rw = (class_rw_t *)calloc(sizeof(class_rw_t), 1);
    rw->ro = ro;
    rw->flags = RW_REALIZED|RW_REALIZING;
    cls->setData(rw);
  }

  // 其它的一些初始化

  // Attach categories
  methodizeClass(cls);
}

static void methodizeClass(Class cls){
  // 将ro中的基本方法添加到rw中去
  method_list_t *list = cls->data()->ro->baseMethods();
  prepareMethodLists(cls, &list, 1, YES, isBundleClass(cls));
  cls->data()->methods.attachLists(&list, 1);

  // 初始化属性、协议和分类
}
```

## 向类动态添加方法

`class_addMethod` 这个函数可以在运行期对类动态添加方法，我们来看下是怎么实现的：

```
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types){
   return !addMethod(cls, name, imp, types ?:"", NO);
}

static IMP addMethod(Class cls, SEL name, IMP imp, const char *types, bool replace){
  method_t *m;
  IMP result;
  if((m = getMethodNoSuper_nolock(cls, name)){
    // 已经存在这个方法
    if(!replace){
      result = m->imp;
    }else{
      result = _method_setImplementation(cls, m, imp); // 重新设置imp
    }
  }else{
    // 添加方法
    method_list_t *newlist = (method_list_t *)calloc(sizeof(*newlist), 1);
    newlist->first.name = name;
    newlist->first.types = types;
    newlist->first.imp = imp;
    prepareMethodsLists(cls, &newlist, 1, NO, NO);
    cls->data()->methods.attachLists(&newlist, 1);
    flushCaches(cls);

    result = nil;
  }

  return resul;
}
```
向一个类中添加新方法，cls->data()->methods.attachLists(&newlist, 1)，仍然是添加到class_rw_t.methods列表中。



