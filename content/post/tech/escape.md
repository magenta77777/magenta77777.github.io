---
title: "逃逸分析"
date: 2023-03-14T14:37:06+08:00
lastmod: 2023-03-14T14:37:06+08:00
author: ["bmlv9909"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- Golang
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
math: true
cover:
    image: "img/escape.png" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---


## 1 逃逸分析

### 1.1 逃逸分析

逃逸分析：分析指针动态范围的方法。当一个对象的指针被多个方法或线程引用时，我们称这个指针发生了逃逸。Go 语言的逃逸分析是编译器执行静态代码分析后，对内存管理进行的优化和简化，它可以决定一个变量分配到堆还栈上。因此 Go 语言中 new 的对象不一定就在堆上，而是由编译器决定。

+ 基本原则：**如果一个函数返回对一个变量的引用，那么它就会发生逃逸**。栈中只保存函数结束后不被引用的对象。
+ 变量取址：可能会被分配到堆上，但是如果编译器发现该变量在**函数结束后**不被引用，则仍然分配到栈上。

根据外部引用决定是否逃逸：

> 1. 如果函数外部没有引用，则优先放到栈中；
> 2. 如果函数外部存在引用，则必定放到堆中；

******



### 1.2 内存分析

逃逸分析可以为变量合理地选择堆和栈。即使是 new 申请的内存也可能分配到栈上；即使表面上只是一个普通变量，也可能最终被分配到堆上。**这十分有利于 GC 的工作**。下面是堆和栈的对比：

| 堆                         | 栈                             |
| -------------------------- | ------------------------------ |
| 适合不可预知大小的内存分配 | 适合固定大小的内存分配         |
| 分配速度慢                 | 分配速度快                     |
| 需要 GC 回收，占用系统资源 | 随函数调用自动回收             |
| 依赖 GC 回收               | 使用 CPU PUSH/RELEASE 指令回收 |

通过逃逸分析，可以尽量把那些不需要分配到堆上的变量直接分配到栈上，堆上的变量少了，会减轻分配堆内存的开销，同时也会减少 GC 的压力，提高程序的运行速度。

******



### 1.3 示例

> 如何查看逃逸分析结果？

+ `go build -gcflags '-m -l' main.go` 查看逃逸分析结果。
+ 汇编代码，查看变量的分配情况。



> 逃逸分析

例一：返回的是 x 的值传递，不发生逃逸。

```go
package main
type S struct {}

func main() {
  var x S
  _ = identity(x)
}

func identity(x S) S {
  return x
}
```

例二：没有对 `identify` 中的 `z` 取引用，因此不发生逃逸。

```go
package main

type S struct {}

func main() {
  var x S
  y := &x
  _ = *identity(y)
}

func identity(z *S) *S {
  return z
}
```

例三：尽管对于 `ref` 没有后续的使用，但在该函数中确实发生了对 `z` 的取引用，因此 `z` 发生逃逸。

```go
package main

type S struct {}

func main() {
  var x S
  _ = *ref(x)
}

func ref(z S) *S {
  return &z
}
```

例四：对 y 取引用，发生逃逸。

```go
package main

type S struct {
  M *int
}

func main() {
  var i int
  refStruct(i)
}

func refStruct(y int) (z S) {
  z.M = &y
  return z
}
```

例五：尽管对 `i` 取引用，但 `z` 最终作为返回值，作用域没有超出 `main`，因此不发生逃逸。

```go
package main

type S struct {
  M *int
}

func main() {
  var i int
  refStruct(&i)
}

func refStruct(y *int) (z S) {
  z.M = y
  return z
}
```

例六：`z` 不再作为返回值，因此 `i` 发生逃逸。

```go
package main

type S struct {
  M *int
}

func main() {
  var x S
  var i int
  ref(&i, &x)
}

func ref(y *int, z *S) {
  z.M = y
}
```





























