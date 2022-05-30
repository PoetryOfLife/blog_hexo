---
title: HTML基础
date: 2016/3/12 19:13:30
tags: 
 - HTML
 - 前端
categories: 
 - 前端
---


# 前言

概述：HTML由标签组成，在学习HTML的时候，记录一些基础常用标签

<!-- more -->

## 一、标签  

### 1.标题

``` HTML
<h1>This is a heading</h1>
<h2>This is a heading</h2>
<h3>This is a heading</h3>
```  

### 2.段落

``` HTML
<p>This is a paragraph.</p>
```

### 3.链接

``` HTML
<a href="http://www.baidu.com">This is a link</a>
```

### 4.图像

``` HTML
<img src="" width="" height="" />
```

### 5.水平线

``` HTML
<hr />
```

### 6.注释

``` HTML
<!-- This is a comment -->
```

### 7.折行

``` HTML
<br />
```

### 8.表格

``` HTML
<table border="1">
<tr>
<td>row 1, cell 1</td>
<td>row 1, cell 2</td>
</tr>
<tr>
<td>row 2, cell 1</td>
<td>row 2, cell 2</td>
</tr>
</table>
```

### 9.无序列表

``` HTML
<ul>
<li>Coffee</li>
<li>Milk</li>
</ul>
```

### 10.有序列表

``` HTML
<ol>
<li>Coffee</li>
<li>Milk</li>
</ol>
```

### 11.块元素

``` HTML
<div> 定义文档中的分区或节（division/section）。
<span> 定义 span，用来组合文档中的行内元素。
```

## 二、属性

HTML 标签可以拥有属性。属性提供了有关 HTML 元素的更多的信息。
属性总是以名称/值对的形式出现，比如：name="value"。
属性总是在 HTML 元素的开始标签中规定。

例：

``` HTML
<a href="http://www.baidu.com">This is a link</a>
```

### 1. style 属性用于改变 HTML 元素的样式

``` HTML
<body>
<h1 style="font-family:verdana">A heading</h1>
<p style="font-family:arial;color:red;font-size:20px;">A paragraph.</p>
</body>
```

### 2. id 属性

id 属性指定 HTML 元素的唯一 ID。 id 属性的值在 HTML 文档中必须是唯一的。
id 属性用于指向样式表中的特定样式声明。JavaScript 也可使用它来访问和操作拥有特定 ID 的元素。
id 的语法是：写一个井号 (#)，后跟一个 id 名称。然后，在花括号 {} 中定义 CSS 属性。

## 三、类

对 HTML 进行分类（设置类），使我们能够为元素的类定义 CSS 样式。
为相同的类设置相同的样式，或者为不同的类设置不同的样式。

``` HTML
<!DOCTYPE html>
<html>
<head>
<style>
.cities {
    background-color:black;
    color:white;
    margin:20px;
    padding:20px;
} 
</style>
</head>

<body>

<div class="cities">
<h2>London</h2>
<p>
London is the capital city of England. 
It is the most populous city in the United Kingdom, 
with a metropolitan area of over 13 million inhabitants.
</p>
</div> 

</body>
</html>
```

## 四、header 头部

### 1. `<head>` 元素

`<head>` 元素是所有头部元素的容器。`<head>` 内的元素可包含脚本，指示浏览器在何处可以找到样式表，提供元信息，等等。

### 2. `<title>` 元素

`<title>` 标签定义文档的标题。

### 3. `<base>` 元素

`<base>` 标签为页面上的所有链接规定默认地址或默认目标

### 4. `<link>` 元素

`<link>` 标签定义文档与外部资源之间的关系

### 5. `<style>` 元素

`<style>` 标签用于为 HTML 文档定义样式信息

### 6. `<meta>` 元素

元数据（metadata）是关于数据的信息。
`<meta>` 标签提供关于 HTML 文档的元数据。元数据不会显示在页面上，但是对于机器是可读的

### 7. `<script>` 元素

`<script>` 标签用于定义客户端脚本，比如 JavaScript
