---
title: vue基础知识
date: 2021/6/05 22:11:23
tags: 
 - 前端
 - Vue
categories: 
 - 前端
 - Vue
---

## 前言

概述：Vue是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。另一方面，Vue 完全有能力驱动采用单文件组件和 Vue 生态系统支持的库开发的复杂单页应用。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。框架是一套现成的解决方案，只能遵守框架的规范，去编写自己的业务功能。
<!-- more -->

## 特性

1. 数据驱动视图

2. 双向数据绑定

## 使用步骤

1. 导入vue.js脚本文件
2. 声明被vue控制的区域
3. 创建vm实例化对象

## 一、 vue实例

el：挂载点
作用范围：vue会管理el选项中命中的元素及其内部的后代元素
data：数据源  
渲染到页面的数据，复杂数据遵守JS语法即可
methods:事件处理函数

```js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  },
  methods:{
    doSomething(){

    }
  }
})
```

## 二、指令

指令 (Directives) 是带有 v- 前缀的特殊 attribute。指令 attribute 的值预期是单个 JavaScript 表达式 (v-for 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM，辅助开发者渲染页面。

### 1. 内容渲染指令

辅助开发者渲染DOM元素的文本内容

#### 1.1 v-text

缺点：会覆盖原来的内容

```HTML  
<p v-text = "username"></p>

```  

#### 1.2 {{}}

数据绑定最常见的形式就是使"Mustache"语法 (双大括号) 的文本插值

```HTML  
<!-- 插值表达式 -->
<span>Message: {{ msg }}</span> 
```  

#### 1.3 v-html

可以把带有标签的字符串渲染成真正的HTML
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：

```HTML
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

### 2. 属性绑定指令

注意：插值表达式只能用在内容节点中，不能用在属性节点中

#### 2.1 v-bind

为元素的属性动态绑定值

```HTMl
<a v-bind:href="url">...</a>

<!-- v-bind 可以简写成: -->
<a :href="url">...</a>

<!-- 动态参数 -->
<a v-bind:[attributeName]="url"> ... </a>
```

使用 JavaScript 表达式
迄今为止，在我们的模板中，我们一直都只绑定简单的 property 键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持
如果绑定内容需要进行动态拼接，则字符串外面需要包裹一层单引号

```HTML
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

### 3. 事件绑定指令

#### 3.1 v-on

为DOM元素绑定事件监听

```HTML
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

#### 3.2 事件修饰符

- 使用this能够访问数据源的数据
- 事件绑定可以在方式中传一个入参e，调用方不传参数，或者传入$event
- 事件修饰符 (modifier) 是以半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定  
例如，.prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()：

```
<form v-on:submit.prevent="onSubmit">...</form>
```

#### 3.3 按键修饰符

在监听键盘事件时，我们经常需要判断详细的按键，此时，可以为键盘相关事件添加按键修饰符
