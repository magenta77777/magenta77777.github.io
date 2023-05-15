---
title: "Problem"
date: 2023-03-14T20:12:55+08:00
lastmod: 2023-03-14T20:12:55+08:00
author: ["bmlv9909"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- 
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
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---


## defer

### 多个 defer

栈方式执行，最后 defer 的操作最先执行。



### return 与 defer

1. 为 return 指定返回的实数值或地址。
2. 执行 defer 指令。
3. 执行 RET 命令，最终返回值。

```go
// 不带命名返回值
func test() int {
  	i := 0
  	defer func() {
      	i++
      	fmt.Println("ans2: ", i)
    }()
  	defer func() {
      	i++
      	fmt.Println("ans1: ", i)
    }()
  	return i
}
// ans1: 1
// ans2: 2
// return 0
/*****************************************************/
// 带命名返回值
func test() (i int) {
  	defer func() {
      	i++
      	fmt.Println("ans2: ", i)
    }()
  	defer func() {
      	i++
      	fmt.Println("ans1: ", i)
    }()
  	return i
}
// ans1: 1
// ans2: 2
// return 2
```

******



## make 与 new

new 非配内存，并不初始化，默认设为 0 值，可以被直接使用。返回指向这段地址的指针。

make 只能够初始化 Slice， map，channel 三种数据结构，因为这三个的底层结构体中的部分字段(len, cap 等)需要被初始化。

******



## 空结构体

struct{}不占据任何内存空间

一般有三个用途：

1. 实现set集合
2. 和channel配合使用，不具备任何意义，但除用作goroutine之间通知
3. 实现一个不带字段，仅包含方法的结构体

******



## 进程、线程、协程

进程：操作系统分配资源的最小单位，拥有独立的寄存器、栈、内存空间等上下文资源。

线程：程序运行的最小单位，同一进程可能拥有多个线程，每个占有虚拟内存空间，共享进程的资源。

协程：更小的单位，有调度器实现。

+ 大小：协程约为 2K，可以动态扩容；线程约 2M。
+ 切换：协程完全在用户态实现；线程需要用户态和内核态的切换。
+ 调度：协程由 runtime 调度器完成；线程由操作系统调度。














