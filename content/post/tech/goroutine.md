---
title: "Goroutine"
date: 2023-03-10T19:34:17+08:00
lastmod: 2023-03-10T19:34:17+08:00
author: ["bmlv9909"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- Golang
- goroutine
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
    image: "img/goroutine.png" #图片路径例如：posts/tech/123/123.png
    zoom:  # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

## 1 goroutine 的实现

### 1.1 GPM 模型

GPM 是 Go 调度器的三个核心组件。

#### 1.1.1 Goroutine

G：Goroutine，有自己的栈、指向堆的指针，保存状态信息与寄存器值，用于当前 goroutine 被调度时使用。

需要注意的是，`G` 中还关联了两个结构体，分别为 stack 和 gobuf。

```go
// 描述栈的数据结构，栈的范围：[lo, hi)
type stack struct {
    // 栈顶，低地址
    lo uintptr
    // 栈低，高地址
    hi uintptr
}
```

```go
// 存储 goroutine 运行时寄存器的值
type gobuf struct {
    // 存储 rsp 寄存器的值
    sp   uintptr
    // 存储 rip 寄存器的值
    pc   uintptr
    // 指向 goroutine
    g    guintptr
    ctxt unsafe.Pointer // this has to be a pointer so that gc scans it
    // 保存系统调用的返回值
    ret  sys.Uintreg
    lr   uintptr
    bp   uintptr // for GOEXPERIMENT=framepointer
}
```



#### 1.1.2 Machine

M：Machine，内核级线程，goroutine 需要调度到 M 上才能运行，M 是真正执行 G 的组件，维护 M 的栈信息、正在执行的 G 信息等。



#### 1.1.3 Processor

P：Processor，为 M 的执行提供上下文，保存 M 执行 G 的资源，维护需要被执行的处于 runnable 状态的 goroutine 队列。

+ LRQ：本地可用 goroutine 队列。
+ GRU：全局可用 goroutine 队列。

当 P 发现 LRQ 为空时，会检查 GRQ 是否为空，不为空则从中获取 G 运行；否则从其他 P 中“偷取” G 执行。



#### 1.1.4 工作模型

每次 go func，都会有一个新的协程被添加到队列结尾。

<center><img src="http://magenta-note-1305707521.coscd.myqcloud.com/62031928-02a8f880-b21b-11e9-96a9-96820452463e.png" alt="62031928-02a8f880-b21b-11e9-96a9-96820452463e" style="zoom:20%;" /><img src="http://magenta-note-1305707521.coscd.myqcloud.com/1075473-20180704160300058-287296807.jpg" alt="1075473-20180704160300058-287296807" style="zoom:50%;" /></center>

// TODO：补充 GPM 的内容

******



### 1.2 goroutine 与 thread

| Goroutine                         | Thread                    |
| --------------------------------- | ------------------------- |
| Goroutine 由 go runtime 管理      | thread 由 kernal 管理。   |
| Goroutine 是用户级的              | thread 是内核级的         |
| Goroutine 有可生长的分段堆栈。    | thread 没有可生长的堆栈。 |
| goroutine 只需要保存三个寄存器    | thread 切换保存多种寄存器 |
| goroutine 切换更快                | thread 切换慢             |
| goroutine 间可以通过 channel 通信 | thread 间通信延迟高       |

******



### 1.3 go scheduler

Go 程序的执行分为两层：Go program 和 runtime，即用户程序和运行时，它们通过函数调用来实现内存管理、channel 通信、goroutine 创建等功能。用户程序进行系统调用会被 runtime 拦截，协助其进行调度与 GC 执行工作。

**runtime 维护所有 goroutine，并通过 scheduler 进行调度**

1. go scheduler 使用 M:N 模型，在任一时刻，M 个 G(oroutine) 分配到 N 个 M(achine) 中，在若干个 P 上运行。
2. 每个 M 都必须依附于一个 P，每个 P 可以挂载多个 M，同时只能运行一个 M。
3. go schedule 每一轮调度都需要找到一个 runnable 的 G，并执行。

goroutine 是 go runtime 的一部分，与 go 程序一起运行，不受用户控制。

******



## 2 goroutine 的细节

### 2.1 调度时机

在以下四种情况，goroutine **可能**发生调度。

+ 使用 go 关键字：创建一个新的 goroutine，Go scheduler 可能调度。
+ GC：GC 工作时也依赖 goroutine 进行调度。
+ 系统调用：当 goroutine 进行系统调用时，会阻塞 M，此时被调度走；同时一个新的 goroutine 会被调度上来。
+ 内存同步访问：atomic，mutex，channel 操作等会使 goroutine 阻塞，此时被调度走；等条件满足后（例如其他 goroutine 解锁了）会被调度上来继续运行。














