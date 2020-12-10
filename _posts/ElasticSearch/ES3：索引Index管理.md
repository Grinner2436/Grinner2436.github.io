---
layout: post
title: 索引管理
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 索引管理
---
# 索引管理

### 一、删除索引
#### 1. 删除语句
```
# ID
DELETE twitter

# 多值
DELETE twitter,facebook

# 全部
DELETE *
DELETE _all
```

#### 2. 批量删除控制
```
PUT _cluster/settings
{
    "persistent": {
        "action.destructive_requires_name": "true" 
    }
}
true:不允许批量删除，必须制定具体索引名
false：可以通过_all和*批量删除
```
###  二、查看索引信息
#### 1. 查询语句
```
# ID
GET twitter

# 多值
GET twitter,facebook

# 全部
GET *
GET _all
```
###  三、索引是否存在
判断索引或者别名是否存在，无法具体区分出是索引还是别名。
#### 1. 查询语句
```
HEAD  twitter
```
#### 2. 响应格式

```
不存在：
404 - Not Found
存在：
200 - OK
```
###  四、打开关闭索引
只要索引处于open状态，就会占用内存+磁盘，如果将索引close，只会占用磁盘。
#### 1. 查询语句
```
# 控制一个
POST  /my_index/_close  
POST  /my_index/_open

# 控制全部
POST  /_all/_open
POST  /*/_open
依然是action.destructive_requires_name控制开关的，见上文。
```
#### 2. 功能开关
```
PUT _cluster/settings
{
    "persistent": {
        "cluster.indices.close.enable": "true" 
    }
}
true:可以使用开关索引功能API
false：禁止使用开关索引功能API
```