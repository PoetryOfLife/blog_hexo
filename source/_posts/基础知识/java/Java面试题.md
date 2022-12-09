---
title: Java面试题
date: 2022/07/01 09:42:51
tags:
  - Java
categories:
  - Java
---

## Java基础

### 1.JDK和JRE有什么区别？

JDK 是java开发工具包，提供了java的开发环境
JRE 是java运行环境，为java的运行提供了所需环境

### 2.== 和 equals 的区别

== 对于基本类型来说是值比较，对于引用类型来说比较的是引用
equals 本质也是==，默认情况下是引用比较，只是在很多类中重写了equals方法，比如String，Integer等把它变成了值比较，所以一般情况下是是值比较

### 3
