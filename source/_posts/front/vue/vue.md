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

# 前言

概述：Vue是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。另一方面，Vue 完全有能力驱动采用单文件组件和 Vue 生态系统支持的库开发的复杂单页应用。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。
<!-- more -->
## 一、 vue实例

el：挂载点
作用范围：vue会管理el选项中命中的元素及其内部的后代元素
data：数据源
复杂数据遵守JS语法即可

```js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

## 指令

指令 (Directives) 是带有 v- 前缀的特殊 attribute。指令 attribute 的值预期是单个 JavaScript 表达式 (v-for 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

1. 参数
一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，v-bind 指令可以用于响应式地更新 HTML attribute：

```HTMl
<a v-bind:href="url">...</a>
```

2. 动态参数
方括号括起来的 JavaScript 表达式作为一个指令的参数：

```HTML
<!--注意，参数表达式的写法存在一些约束，如之后的“对动态参数表达式的约束”章节所述。-->
<a v-bind:[attributeName]="url"> ... </a>
```

3. 修饰符
修饰符 (modifier) 是以半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定  
例如，.prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()：

```HTML
<form v-on:submit.prevent="onSubmit">...</form>
```

## 插值

1. 文本 v-text
    数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值

```HTML
<span>Message: {{ msg }}</span>
```

2. 原始 HTML v-html
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：

```HTML
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

3. Attribute
Mustache 语法不能作用在 HTML attribute 上，遇到这种情况应该使用 v-bind 指令

```HTML
<div v-bind:id="dynamicId"></div>
```

4. 使用 JavaScript 表达式

迄今为止，在我们的模板中，我们一直都只绑定简单的 property 键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持

```HTML
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

5. v-show
通过值判断显示还是不显示，只改变display

6. v-if
根据表达式的真假，切换元素的显示还是隐藏，直接移除标签

7. v-bind
设置元素属性

8. v-for
根据数据生成列别

9. v-model
获取和设置表单元素（双向数据绑定）

## 缩写

1. v-bind 缩写

```HTML
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a :[key]="url"> ... </a>
```

2. v-on 缩写

```HTML
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

## axios

网络请求库

``` js
GET：axios.get(url+key=value&key=value).then(function(response){},function(error){})
POST：axios.post(url,{key:value,key:value}).then(function(response){},function(error){})
```
