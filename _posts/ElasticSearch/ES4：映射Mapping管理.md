---
layout: post
title: Mapping管理
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: Mapping管理
---
# ES4：映射Mapping管理

### 添加字段

```
PUT /twitter-1,twitter-2/_mapping/_doc 
{
  "properties": {
    "user_name": {
      "type": "text"
    }
  }
}
```
（1）可以添加新字段
（2）可以为多个索引同时添加字段，并且可以使用通配符

### 更新属性
默认Mapping里的各项属性是不能修改的，以下是特例：
* Object类型可以增加新字段
* 多类型字段，可以扩充新类型
* `ignore_above`参数的值可以被改变

```
PUT my_index/_mapping/_doc
{
  "properties": {
    "name": {
      "properties": {
        "last": { 
          "type": "text"
        }
      }
    },
    "user_id": {
      "type": "keyword",
      "ignore_above": 100 
    }
  }
}
```
### 查询映射
获取映射的设置
```
GET /_mapping/_doc
GET /_all/_mapping/_doc
```
获取所有索引的字段映射
```
GET /_all/_mapping
GET /_mapping
```
以字段挑选部分映射
```
GET /twitter,kimchy/_mapping/field/message

GET /_all/_mapping/_doc/field/message,user.id

GET /_all/_mapping/_doc/field/*.id
```


