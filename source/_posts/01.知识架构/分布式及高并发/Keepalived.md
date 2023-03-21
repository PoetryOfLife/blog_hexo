---
title: Keepalived
date: 2022/06/23 11:23:33
tags: 
 - Keepalived
categories: 
 - Keepalived
---



# 一、简介

Keepalived一个基于VRRP协议来实现的 LVS 服务高可用方案，可以利用其来解决单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个`虚拟IP`，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。

<!-- more -->

# 二、Keepalived 的作用

如上述所说，**Keepalived** 提供了很好的`高可用性保障服务`，它可以检查服务器的状态，如果有服务器出现问题，Keepalived 会将其从系统中移除，并且同时使用备份服务器代替该服务器的工作，当这台服务器可以正常工作后，Keepalived 再将其放入服务器群中，这个过程是 Keepalived 自动完成的，不需要人工干涉，我们只需要修复出现问题的服务器即可。

# 三、Keepalived 原理

## 3.1 基于VRRP协议的理解

Keepalived 是以 `VRRP` 协议为实现基础的，VRRP全称`Virtual Router Redundancy Protocol`，即`虚拟路由冗余协议`。

虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master 和多个 backup，master 上面有一个对外提供服务的 `VIP(Virtual IP Address)`（该路由器所在局域网内其他机器的默认路由为该 vip），master 会发组播，当 backup 收不到 vrrp 包时就认为 master 宕掉了，这时就需要根据 VRRP 的优先级来`选举`一个 backup 当 master。这样的话就可以保证路由器的高可用了。

keepalived 主要有三个模块，分别是core、check 和 vrrp。core 模块为keepalived的**核心**，负责主进程的启动、维护以及全局配置文件的加载和解析。check 负责健康检查，包括常见的各种检查方式。vrrp 模块是来实现 VRRP 协议的。

### 3.2 基于TCP/IP协议的理解

以检测 web 服务器为例，Keepalived 从3个层次来检测服务器的状态

Layer3 、Layer4 以及 Layer7 工作在IP/TCP协议栈的IP层，TCP层，及应用层,原理分别如下：

#### 3.2.1 **Layer3**

Keepalived使用Layer3的方式工作时，Keepalived会定期向服务器群中的服务器发送一个ICMP的数据包（既我们平时用的Ping程序）,如果发现某台服务的IP地址没有激活，Keepalived 便报告这台服务器失效，并将它从服务器群中剔除，这种情况的典型例子是某台服务器被非法关机。Layer3 的方式是以服务器的`IP地址`是否有效作为服务器工作正常与否的标准。

#### 3.2.2 **Layer4**

如果您理解了Layer3的方式，Layer4就容易了。Layer4主要以`TCP 端口的状态`来决定服务器工作正常与否。如 web server 的服务端口一般是80，如果 Keepalived 检测到80端口没有启动，则 Keepalived 将把这台服务器从服务器群中剔除。

#### 3.2.3 **Layer7：**

Layer7 就是工作在具体的应用层了，比Layer3,Layer4要复杂一点，在网络上占用的带宽也要大一些。Keepalived 将根据用户的设定检查服务器程序的运行是否正常，如果与用户的设定不相符，则 Keepalived 将把服务器从服务器群中剔除。



![img](https://pic1.zhimg.com/v2-2784a5063dfcd3043c3a45b27c1e9ac4_b.jpg)



# 四、Keepalived 选举策略

## 4.1 选举策略

首先，每个节点有一个初始优先级，由配置文件中的`priority`配置项指定，MASTER 节点的 priority 应比 BAKCUP 高。运行过程中 keepalived 根据 vrrp_script 的 `weight` 设定，增加或减小节点优先级。规则如下：

1. weight值为正时,脚本检测成功时”weight”值会加到”priority”上,检测失败时不加
   - 主失败: 主priority <  备priority+weight之和时会切换
   - 主成功: 主priority+weight之和 > 备priority+weight之和时,主依然为主,即不发生切换
2. weight为负数时,脚本检测成功时”weight”不影响”priority”,检测失败时,Master节点的权值将是“priority“值与“weight”值之差
   - 主失败: 主priotity-abs(weight) < 备priority时会发生切换
   - 主成功: 主priority > 备priority 不切换
3. 当两个节点的优先级相同时，以节点发送`VRRP通告`的 IP 作为比较对象，IP较大者为MASTER。

## 4.2 priority 和 weight 的设定

1. 主从的优先级初始值priority和变化量weight设置非常关键，配错的话会导致无法进行主从切换。比如，当MASTER初始值定得太高，即使script脚本执行失败，也比BACKUP的priority + weight大，就没法进行VIP漂移了。
2. 所以priority和weight值的设定应遵循: abs(MASTER priority - BAKCUP priority) < abs(weight)。一般情况下，初始值MASTER的priority值应该比较BACKUP大，但不能超过weight的绝对值。 另外，当网络中不支持多播(例如某些云环境)，或者出现网络分区的情况，keepalived BACKUP节点收不到MASTER的VRRP通告，就会出现脑裂(split brain)现象，此时集群中会存在多个MASTER节点。
