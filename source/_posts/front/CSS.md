---
title: CSS基础
date: 2016/3/17 14:24:53
tags: 
 - CSS
 - 前端
categories: 
 - 前端
---

# 前言

概述：CSS 是一种描述 HTML 文档样式的语言，描述应该如何显示 HTML 元素

<!-- more -->

## 一、选择器

1. 简单选择器

- 元素选择器

```CSS
p {
  text-align: center;
  color: red;
}
```

- id 选择器

 ```CSS
#para1 {
  text-align: center;
  color: red;
}
```

- 类选择器

 ```CSS
.center {
  text-align: center;
  color: red;
}
```

- 通用选择器

 ```CSS
* {
  text-align: center;
  color: blue;
}
```

- 分组选择器

```CSS
h1 {
  text-align: center;
  color: red;
}

h2 {
  text-align: center;
  color: red;
}

p {
  text-align: center;
  color: red;
}
```

## 二、背景

- background-color 属性指定元素的背景色
- background-image 属性指定用作元素背景的图像。
- background-repeat 默认情况下，background-image 属性在水平和垂直方向上都重复图像。
- background-attachment 属性指定背景图像是应该滚动还是固定的（不会随页面的其余部分一起滚动）：
- background-position background-position 属性用于指定背景图像的位置。
- opacity 属性指定元素的不透明度/透明度。取值范围为 0.0 - 1.0。值越低，越透明

## 三、边框

1. 边框属性 border
2. 边框样式 border-style
3. 边框宽度 border-width 属性指定四个边框的宽度
4. 边框颜色 border-color 属性用于设置四个边框的颜色
5. 单独边
6. 圆角边框 border-radius 属性用于向元素添加圆角边框

```CSS
p {
  border-top-style: dotted;
  border-right-style: solid;
  border-bottom-style: dotted;
  border-left-style: solid;
}
```

## 四、边距

margin 属性用于在任何定义的边框之外，为元素周围创建空间并
当两个垂直外边距相遇时，它们将形成一个外边距，合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者

padding 属性用于在任何定义的边界内的元素内容周围生成空间

轮廓是在元素周围绘制的一条线，在边框之外，以凸显元素。

## 五、文本

color 属性用于设置文本的颜色
text-align 属性用于设置文本的水平对齐方式
text-decoration 属性用于设置或删除文本装饰

## 六、布局

display 属性规定是否/如何显示元素
position 属性规定应用于元素的定位方法的类型
float 属性规定元素如何浮动
clear 属性规定哪些元素可以在清除的元素旁边以及在哪一侧浮动
