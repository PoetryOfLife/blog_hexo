---
title: WebPack自动化构建
date: 2021/6/01 23:54:43
tags: 
 - 前端
 - Vue
categories: 
 - 前端
 - Vue
---

# 前言

概述：webpack实现前端项目的模块化，旨在更高效地管理和维护项目中的每一个资源
<!-- more -->
## 1、项目工程化

1. 模块化 （js、css的模块化）
2. 组件化 复用现有的UI结构，样式，行为
3. 规范化 目录结构、编码规范、接口、git
4. 自动化 自动化构建、部署、测试

## 2、webpack

前端工程化的具体解决方案，提供了友好的前端模块化开发，以及代码压缩混淆、处理浏览器JavaScript的兼容行、性能优化等功能

### 2.1 安装

```bash
npm install webpack@5.42.1 webpack-cli@4.7.2 -D
```

在package.json中
dependencies - 生产需要的包，命令中添加-S
devDependencies - dev环境需要的生产不需要的包，命令中添加-D

### 2.2 配置

1. 在根目录中中创建webpack.config.js，并初始化如下配置：

```js
module.exports = {
    mode: "development"
}
```

2. 在package.json的scripts节点下，新增dev和build
scripts：可运行脚本

```js
  "scripts": {
    "dev": "webpack server",
    "build": "webpack --mode production"
  }
```

1. 在终端数据npm run dev完成打包构建

### 2.3 指定entry和output

默认情况下，打包入口文件为：src -> index.js
默认情况下，打包输出文件为：dist -> main.js

在 webpack.config.js 文件中，通过entry节点指定打包的入口，通过output节点指定打包的出口。

```js
module.exports = {
    mode: "development",

    //指定处理哪个文件
    entry: path.join(__dirname, "./src/index.js"),
    //指定生成文件存放目录
    output: {
        path: path.join(__dirname, "dist"),
        filename: "bundle.js"
    }
}

```

### 2.4  devServer常用选项

```js
module.exports = {
    mode: "development",
    plugins: [htmlPlugin],
    devServer: {
        open: true, // 自动打开浏览器
        port: 8081, // 修改端口
        host: '127.0.0.1' // 指定地址
    }
}
```

## 3. webpack-dev-server插件

热更新，自动生成输出文件，安装命令如下

### 3.1 安装

```bash
npm install webpack-dev-server@3.11.2 -D
```

### 3.2 配置

1. 修改package.json中的配置如下，在webpack后加上server

```json
  "scripts": {
    "dev": "webpack server"
  },
 ```

2. 再次运行npm run dev命令

3. 如果启动失败，遇到以下问题：
![启动失败](../../../image/webPack_1.png)  
则需要额外安装

```bash
npm i --save-dev webpack-cli  -D
```

### 3.3 工作原理

webpack会把输出文件放在内存中，所以可以在保存文件后进行打包编译，实现热更新

## 4. html-webpack-plugin插件

### 4.1 安装

```bash
npm install html-webpack-plugin@5.3.2 -D
```

### 4.2 配置

修改package.json中的配置如下，在webpack后加上server

```js
//导入插件
const HtmlPlugin = require("html-webpack-plugin");
//创建实例
const htmlPlugin = new HtmlPlugin({
    //源文件
    template: './src/index.html',
    //被复制的文件
    filename: './index.html'
})
//使用插件
module.exports = {
    mode: "development",
    plugins: [htmlPlugin]
}
 ```

### 4.3 特性

1. 将指定的html文件复制到指定目录下
2. 注入生成的js文件

## 5. loader

webpack只能打包js结尾的文件，其他非js结尾的文件由loader打包特定文件，比如：css-loader  
当webpack发现某个文件不能处理时，回去找相应的loader处理

### 5.1 style-loader、css-loader

#### 5.1.1 安装

```bash
npm install style-loader@3.0.0 css-loader@5.2.6 -D
```

#### 5.1.2 配置

在webpack.config.js的module -> rules中配置loader规则

```js
  module: {
      rules: [{
          test: /\.css$/,
          use: ['style-loader', 'css-loader'] // 从后往前调用，依次处理
      }]
  }
 ```

### 5.2 url-loader

#### 5.2.1 安装

```bash
npm install url-loader@4.1.1 file-loader@6.2.0 -D
```

#### 5.2.2 配置

在webpack.config.js的module -> rules中配置loader规则

```js
  module: {
      rules: [{
          test: /\.jpg|png|gif$/,
          use: ['url-loader?limit=22229'] // ?之后是loader的参数项，只有小于limit的图片，才会转换为base64
      }]
  }
 ```

### 5.3 label-loader

转换处理js中webpack不能处理得一些高级语法

#### 5.3.1 安装

```bash
npm install babel-loader@8.2.2 @babel/core@7.14.6 @babel/plugin-proposal-decorators@7.14.5 -D
```

#### 5.3.2 配置

在webpack.config.js的module -> rules中配置loader规则

```js
module: {
    rules: [{
        test: /\.js$/,
        use: ['babel-loader'], // ?之后是loader的参数项，只有小于limit的图片，才会转换为base64
        exlude: /node_modules/ // 排除不需要处理得代码
    }]
}
 ```

在根目录下创建babel.config.js，并在此文件中添加可用插件

```js
module.exports = {
    // 生命babel可用得插件
    plugins: [['@babel/plugin-proposal-decorators', {legacy: true}]]
}

```
