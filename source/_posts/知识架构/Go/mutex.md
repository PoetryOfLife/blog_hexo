---
title: mutex
date: 2022/07/08 19:02:11
tags:
  - Go
categories:
  - Go
---

## 同步锁

当一个 Goroutine（协程）获得了 Mutex 后，其他 Goroutine（协程）就只能乖乖的等待，除非该 Goroutine 释放了该 Mutex。RWMutex 在读锁占用的情况下，会阻止写，但不阻止读。RWMutex在写锁占用情况下，会阻止任何其他Goroutine（无论读和写）进来，整个锁相当于由该 Goroutine 独占同步锁的作用是保证资源在使用时的独有性，不会因为并发而导致数据错乱，保证系统的稳定性。  

<!-- more -->

## Mutex

### 四种状态

- Locked - 互斥锁锁定状态
- Woken - 是否有协程已被唤醒
- Starving - 是否处于饥饿状态
- Waiter - 表示等待锁的协程个数，协程解锁时根据此值来判断是否需要释放信号量

### 两种模式

- normal:正常模式，所有等待锁的goroutine按照FIFO顺序等待，唤醒的goroutine不会直接拥有锁，而是和新请求的goroutine竞争锁，新请求的goroutine更容易竞争到锁，因为它正在CPU上执行，所以刚刚唤醒的goroutine很大可能会竞争失败，这种情况下，会把刚刚唤醒的goroutine放到队列前面，而且加锁不成功会判断是否满足自旋条件，如果满足则会自动自旋，尝试抢锁。
- 饥饿模式 在饥饿模式下，会直接把锁交给队列第一位的goroutine，新进来的G也不会进入自旋状态而是放到队尾，当一个goroutine等待锁时间超过1ms时，或者当前队列只剩下一个goroutine时，Mutex切换到饥饿模式。

### 自旋条件

- 锁已经被占用了，并且锁不处于饥饿模式
- 积累的自选次数小于4次
- CPU核数大于1
- 有空闲的P
- 当前goroutine所挂载的P下，本地队列为空
