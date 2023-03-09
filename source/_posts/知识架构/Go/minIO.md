---
title: minIO
date: 2021/03/12 23:11:59
tags:
  - minIO
categories:
  - minIO
---

# 一、简介

minIO 是一个基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。minIO 是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL。

<!-- more -->

# 二、特点

- 高性能：作为高性能对象存储，在标准硬件条件下它能达到55GB/s的读、35GG/s的写速率。
- 可扩容：不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并跨越多个数据中心。
- 云原生：容器化、基于K8S的编排、多租户支持。
- Amazon S3兼容：Minio使用Amazon S3 v2 / v4 API。可以使用Minio SDK，Minio Client，AWS SDK和AWS CLI访问Minio服务器。
- 可对接后端存储: 除了Minio自己的文件系统，还支持DAS、 JBODs、NAS、Google云存储和Azure Blob存储。
- SDK支持: 基于Minio轻量的特点，它得到类似Java、Python或Go等语言的sdk支持。
- Lambda计算: Minio服务器通过其兼容AWS SNS / SQS的事件通知服务触发Lambda功能。支持的目标是消息队列，如Kafka，NATS，AMQP，MQTT，Webhooks以及Elasticsearch，Redis，Postgres和MySQL等数据库。
- 有操作页面
- 功能简单: 这一设计原则让MinIO不容易出错、更快启动。
- 支持纠删码：MinIO使用纠删码、Checksum来防止硬件错误和静默数据污染。在最高冗余度配置下，即使丢失1/2的磁盘也能恢复数据。

# 三、存储机制

## 1. **校验和**

保护数据免受硬件故障和无声数据损坏

## 2. **纠删码**

 纠删码是一种恢复丢失和损坏数据的数学算法，目前，纠删码技术在分布式存储系统中的应用主要有三类，阵列纠删码（Array Code: RAID5、RAID6等）、RS(Reed-Solomon)里德-所罗门类纠删码和LDPC(LowDensity Parity Check Code)低密度奇偶校验纠删码。Erasure Code是一种编码技术，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。 这就意味着如果是12块盘，一个对象会被分成6个数据块、6个奇偶校验块，可以丢失任意6块盘（不管其是存放的数据块还是奇偶校验块），仍可以从剩下的盘中的数据进行恢复。

## 3.**RS code编码数据恢复原理**

RS编码以word为编码和解码单位，大的数据块拆分到字长为w（取值一般为8或者16位）的word，然后对word进行编解码。 数据块的编码原理与word编码原理相同，后文中以word为例说明，变量Di, Ci将代表一个word。把输入数据视为向量D=(D1，D2，..., Dn）, 编码后数据视为向量（D1, D2,..., Dn, C1, C2,.., Cm)，RS编码可视为如下（图1）所示矩阵运算。
 图1最左边是编码矩阵（或称为生成矩阵、分布矩阵，Distribution Matrix），编码矩阵需要满足任意n*n子矩阵可逆。为方便数据存储，编码矩阵上部是单位阵（n行n列），下部是m行n列矩阵。下部矩阵可以选择范德蒙德矩阵或柯西矩阵。

![189732-918cf64c1a46a102](D:\workspace\github\hexo_blog\source\image\189732-918cf64c1a46a102.png)

RS最多能容忍m个数据块被删除。 数据恢复的过程如下：
 （1）假设D1、D4、C2丢失，从编码矩阵中删掉丢失的数据块/编码块对应的行。（图2、3）
 （2）由于B' 是可逆的，记B'的逆矩阵为 (B'^-1)，则B' * (B'^-1) = I 单位矩阵。两边左乘B' 逆矩阵。 （图4、5）
 （3)得到如下原始数据D的计算公式 。

![189732-5d1a7bdb324b0e33](D:\workspace\github\hexo_blog\source\image\189732-5d1a7bdb324b0e33.png)

（4）对D重新编码，可得到丢失的编码

# 四、部署时遇到的问题

## 1. WARNING: Published ports are discarded f using host network mode

![image-20230309101022659](C:\Users\75713\AppData\Roaming\Typora\typora-user-images\image-20230309101022659.png)

使用了--net=host，这个容器使用的实际上是宿主机的ip和端口。

## 2.本机可以访问9090端口，其他机器不行

1.关闭防火墙 2.执行`iptables -F`清空防火墙规则
