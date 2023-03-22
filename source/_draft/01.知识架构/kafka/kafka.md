---
title: kafka
date: 2022/9/25 18:52:25
tags:
  - kafka
categories:
  - kafka 
---


# 一、前言

Kafka已被多家不同类型的公司作为多种类型的数据管道和消息系统使用。行为流数据是几乎所有站点在对其网站使用情况做报表时都要用到的数据中最常规的部分。

<!-- more -->

# 二、基础概念

kafka是一种分布式的，基于发布/订阅的消息系统。主要设计目标如下：

以时间复杂度未O(1)的方式提供消息持久化能力，即使对TB级以上的数据也能保证常数时间复杂度的访问性能；

高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输；

支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消费顺序传输；

同时支持离线数据处理和实时数据处理；

scale out：支持在线水平扩展

## 2.1 Topic主题

Topic在逻辑上可以被认为一个queue，每条消费都必须指定它的Topic，可以简单理解为必须指明它的Topic，可以简单理解为必须指明把这条消息放进哪个queue里。我们把一类消息按照主题来分类，有点类似于数据库中的表。

## 2.2 Broker

Kafka集群包含一个或者多个服务器，每个服务器节点称为一个Broker。

Broker存储Topic的数据。如果某个Topic有N个Partition，集群有N个Broker，那么每个Broker存储该Topic的一个Partition。

从scale out 的性能角度思考，通过Broker Kafka server 的更多节点，带更多的存储，建立更多的Partition 把IO负载到更多的物理节点，提高总吞吐IOPS。

从scale up的角度思考，一个Node拥有越多的Physical Disk，也可以负载更多的Partition，提升总吞吐

Topic只是一个逻辑概念，真正在Broker间分布的时Partition。

# 三、存储原理

Kafka为什么性能好？

Kafka的消息是存在于文件系统之上的。Kafka高度依赖文件系统来存储和缓存消息，一般的人认为”磁盘是缓慢的“。

操作系统还会将主内存剩余的所有空闲内存都用作磁盘缓存，所有的磁盘读写操作都会经过同意的磁盘缓存（除了直接IO会绕过磁盘缓存）。

Kafka利用了顺序IO，在文件尾部追加到数据文件尾部。

利用了linux的page cache的能力。
