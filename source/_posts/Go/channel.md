---
title: channel
date: 2022/07/01 09:42:51
tags:
  - Go
categories:
  - Go
---

## 1. 前言
channel是Golang在语言层面提供的goroutine间的通信方式，比Unix管道更易用也更轻便。channel主要用于进程内各goroutine间通信，如果需要跨进程通信，建议使用分布式系统的方法来解决。

## 2. 数据结构
```
type hchan struct {
	qcount   uint           // 当前队列的剩余元素数量
	dataqsiz uint           // 环形队列长度，即可存放的元素个数
	buf      unsafe.Pointer // 环形队列指针
	elemsize uint16         // 每个元素大小
	closed   uint32         // 关闭状态标识
	elemtype *_type         // 元素类型
	sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
	recvx    uint           // 队列下标，指示元素从队列的该位置读出
	recvq    waitq          // 等待读消息的goroutine队列
	sendq    waitq          // 等待写消息的goroutine队列
  lock     mutex          // 互斥锁，chan不允许并发读写
}
```
### 2.1 环形队列
chan内部实现了一个环形队列作为其缓冲区，队列的长度是创建chan时指定的。

### 2.2 等待队列
从channel

### 实现原理和特性

#### 1. 全局锁

#### 2. 移入、移除元素
