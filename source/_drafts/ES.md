---
title: ElasticSearch
date: 2023/09/15 16:49:21
tags: 
 - ElasticSearch
categories: 
 - ElasticSearch
---
# ElasticSearch

## 一、倒排索引

倒排索引会先把**文档**（document）内容分割成**词条**（term），词条会指向具体的文档ID，这个就可以根据词条快速找到ID，再根据ID找到文档。

## 二、数据类型

文档：一条数据就是一个文档，es是JSON格式。

字段：JSON文档中的字段。

索引：同类型文档的集合。

映射：索引中文档的约束，比如字段名称、类型。

## 三、安装IK分词器

### 作用

创建倒排索引时对文档进行分词；

用户搜索时，对输入的内容分词；

### IK分词器分成两种模式

- ik_max_word 细粒度
- ik_smart 粗粒度

### 拓展和停用词条

IK分词器依赖于字典做分词，ik分词支持拓展词库和停用词库；

在IKAnalyzer.cfg.xml中设置ext.dic和stopword.dic;



