---
title: "Channel"
date: 2023-03-10T21:40:36+08:00
lastmod: 2023-03-10T21:40:36+08:00
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
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---



## Channel 的实现

### 1.1 Channel 的结构

Channel 的底层数据结构为 `hchan`，

```go
type hchan struct {
    // chan 里元素数量
    qcount   uint
    // chan 底层循环数组的长度
    dataqsiz uint
    // 指向底层循环数组的指针
    // 只针对有缓冲的 channel
    buf      unsafe.Pointer
    // chan 中元素大小
    elemsize uint16
    // chan 是否被关闭的标志
    closed   uint32
    // chan 中元素类型
    elemtype *_type // element type
    // 已发送元素在循环数组中的索引
    sendx    uint   // send index
    // 已接收元素在循环数组中的索引
    recvx    uint   // receive index
    // 等待接收的 goroutine 队列
    recvq    waitq  // list of recv waiters
    // 等待发送的 goroutine 队列
    sendq    waitq  // list of send waiters
    // 保护 hchan 中所有字段
    lock mutex
}
```

<img src="http://magenta-note-1305707521.coscd.myqcloud.com/61179068-806ee080-a62d-11e9-818c-16af42025b1b.png" alt="61179068-806ee080-a62d-11e9-818c-16af42025b1b" style="zoom:40%;" />



在实际使用中，Channel 通常有两个方向，即发送和接收，使用 `make` 语句初始化，得到的是 `*hchan` 类型的指针，在堆上分配对应的内存。

```go
ch1 := make(chan int)
ch2 := make(chan int, 10)	// 缓冲型 Channel
```

***



### 1.2 Channel 的接收

接受操作有两种表达式：带 comma 和不带 comma，在底层中执行 `chanrecv` 函数。

// TODO: source code

***



### 1.3 Channel 的发送



******



### 1.4 Select

每个 case 都对应一个 channel，

+ 如果没有 channel 可以执行，执行 default；如果没有 default，阻塞。
+ 如果有一个 channel 可以执行，选择并执行。
+ 如果有多个 channel 可以执行，任意选择一个执行。

```go
select {
 	case <- channel1:
    // 执行的代码
  case value := <- channel2:
    // 执行的代码
  case channel3 <- value:
    // 执行的代码
  default:
}
```



******



### 1.5 context

context 主要用来在 goroutine 之间传递上下文信息，包括取消信号、超时时间、截止时间、k-v 等，常见于并发控制和超时控制，并发安全。

http 请求。

go routine 的取消：

```go
func Perform(ctx context.Context) {
    for {
        calculatePos()
        sendResult()

        select {
        case <-ctx.Done():
            // 被取消，直接返回
            return
        case <-time.After(time.Second):
            // block 1 秒钟 
        }
    }
}
```

增加一个 context，在 break 前调用 cancel 函数，取消 goroutine。gen 函数在接收到取消信号后，直接退出，系统回收资源。

```go
func gen(ctx context.Context) <-chan int {
    ch := make(chan int)
    go func() {
        var n int
        for {
            select {
            case <-ctx.Done():
                return
            case ch <- n:
                n++
                time.Sleep(time.Second)
            }
        }
    }()
    return ch
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // 避免其他地方忘记 cancel，且重复调用不影响

    for n := range gen(ctx) {
        fmt.Println(n)
        if n == 5 {
            cancel()
            break
        }
    }
    // ……
}
```






















