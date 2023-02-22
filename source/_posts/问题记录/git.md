---
title: 问题记录
date: 2022/12/01 22:11:23
tags: 
 - 记录
categories: 
 - 记录
---

## 问题一
### 问题描述
git clone时提示“fatal: remote error: Git repository not found”

### 原因：
下载时，使用的是电脑原同事的git账号

### 解决方案：
win凭据管理中删除原来的凭据即可

## 问题二

### 问题描述

使用 kratos proto server 命令时报错
[!avater]

### 原因
缺少了工具

### 解决方案
手动把上方get包install
go install

## 问题三
### 生成ssh

```shell
cd ~/.ssh
ssh-keygen -t rsa -C "757135670@qq.com"
cat id_rsa.pub
```



## 问题四

### 问题描述

![image-20230216100306598](../../image/%E9%97%AE%E9%A2%98/image-20230216100306598.png)

### 原因

原因是在2022年3月15日之后，github不再支持SHA-1的加密方式了。

### 解决方法

将SHA-1的加密方式修改为`ECDSA`的方式，并把公钥加入到github中，具体操作步骤如下。

1. 生成`ECDSA`密钥

   执行如下命令

   ```shell
   cd ~/.ssh
   ssh-keygen -t ecdsa -b 521 -C "757135670@qq.com"
   ```

2. 重新把公钥放到github的setting中



## 问题五

### 问题描述

在goland中，执行go mod tidy失败，但是手动git clone能够下载代码

### 解决方法

清除提示报错信息的cache目录下的缓存，重新tidy



