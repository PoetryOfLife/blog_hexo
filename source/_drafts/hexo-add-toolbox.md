---
title: hexo+fluid 添加相册功能
comment: valine
tags: 
 - hexo 
categories:
 - hexo
---



### 一、准备

安装 `git`

安装 `python`

### 二、使用

1. 在`github`中创建仓库，用于存储照片，仓库地址如下

```
git@github.com:xxxxxxxx/hexo_album.git
```

将仓库clone 到本地，点击进入文件夹后，创建`photos`及`min_photos`两个文件夹，photos存放你你需要上传的图片。

2. 将图片存放进入photos文件夹，存放名称格式为：`2021-10-15_avatar.jpg`，也可以执行`ImageRename.py`修改名字（需要安装Python环境）

3. 为了快速加载显示图片，我们还需要制作图片的压缩图和图片的序列 data.json.

   下载脚本 ImageProcess.py和tools.py

