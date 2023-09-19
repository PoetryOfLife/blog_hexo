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

## 四、mapping约束

### type

- 字符串：text、keyword
- 数值：long、integer、short、byte、double、float
- 布尔：boolean
- 日期：date
- 对象：object

### index

是否创建索引，默认true

### analyzer

使用哪种分词器

### properties

该字段的子字段



## 五、索引库

### 创建

PUT /索引库名

### 查看

GET /索引库名

### 删除

DELETE /索引库名

### 修改

禁止修改原有字段，但是可以添加新字段名

PUT /索引库名/_mapping

## 六、文档操作

### 新增文档

POST /索引库名/_doc/文档id

### 查询文档

GET /索引库名/_doc/文档id

### 删除文档

DELETE /索引库名/_doc/文档id

### 修改文档

方式一：全量修改，会删除文档，添加新文档

PUT /索引库名/_doc/文档id

方式二：增量修改，修改指定字段值

POST /索引库名/_update/文档id

## 七、DSL

### 查询分类

查询所有：match_all

全文检索：利用分词器对用户的输入进行分词，然后去倒排索引查询。match_query、multi_match_query

精确查询：根据精确词条查询数据，ids，range，term

地理查询：根据经纬度查询。

复合查询：可以将上述各种查询条件组合起来查询。

### 相关性算分

TF（词条频率） = 词条出现次数/词条总数

TF-IDF 算法

![image-20230919105148489](C:\Users\75713\AppData\Roaming\Typora\typora-user-images\image-20230919105148489.png)

DSL 查询语法

### function score query

使用function score query，可以修改文档的相关性算分，根据新得到的算分排序。

![image-20230919164325439](C:\Users\75713\AppData\Roaming\Typora\typora-user-images\image-20230919164325439.png)

三要素：

- 过滤条件：哪些文档要加分
- 算分函数（weight、field_value_factor、random_score、script_score）：如何计算function_score。
- 加权模式（multiply、replace）：query_score和function_score的加权方式。

### 复合查询 boolean query

布尔查询是一个或多个查询子句的组合。子查询组合方式：

- must：必须匹配每个子查询
- should：选择性匹配子查询
- must_not：必须不匹配，不参与算分
- filter：必须匹配

### 搜索结果处理

#### 排序

默认是根据相关度算分来排序，也可以指定字段来排序。

使用sort和field来指定顺序

#### 分页

默认只返回前十条，通过使用from和size来分页，默认最大10000条

ES是分布式的，所以会面临深度分页的问题。

1. 首先从每个数据分片上都排序并查询前1000条。
2. 然后将所有节点的结果聚合，在内存中重新排序选出前1000条文档。
3. 最后从这1000条中，选取从990开始的10条文档。

##### 针对深度分页

search after：分页时需要排序，原来是从上一次的排序值开始，查询下一页数据。

#### 高亮

把搜索结果中把搜索关键字突出显示，通过highlight来实现，其中包裹pre_tags和post_tags属性指定标签

## 聚合

聚合可以实现对文档数据的统计、分析、运算。常见的有三类：

- 桶：用来对文档做分组
  - TermAggregation
  - Date
- 度量聚合：用以计算一些值。
  - avg
  - max
  - min
- 管道聚合：其他聚合的结果进行为基础做聚合。

aggs代表聚合，与query同级，此时query的作用是限定聚合的文档范围。

聚合三要素：聚合名称，聚合类型，聚合字段。

## 自动补全



## 分词器

分词器分为三部分
