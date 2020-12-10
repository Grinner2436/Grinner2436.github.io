---
layout: post
title:  别名Alias
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note:  别名Alias
---
# ES3：别名Alias
## 创建别名

### 1. 为存在的索引创建别名
```
PUT /{index}/_alias/{name}
_alias可以用_aliases替换

PUT /users/_alias/user_12
{
    "routing" : "12",
    "filter" : {
        "term" : {
            "user_id" : 12
        }
    }
}
```
### 2. 为存在的索引关联别名
```
#添加和删除别名
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
#别名多指，多次指向
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
#别名多指，数组方式
POST /_aliases
{
    "actions" : [
        { "add" : { "indices" : ["test1", "test2"], "alias" : "alias1" } }
    ]
}
#别名多指，通配符
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test*", "alias" : "all_test_indices" } }
    ]
}
#别名过滤，部分符合条件的可以被别名查出来
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "test1",
                 "alias" : "alias2",
                 "filter" : { "term" : { "user" : "kimchy" } }
            }
        }
    ]
}
```
### 3. 创建索引的时候关联别名

```
PUT /logs_20162801
{
    "mappings" : {
        "_doc" : {
            "properties" : {
                "year" : {"type" : "integer"}
            }
        }
    },
    "aliases" : {
        "current_day" : {},
        "2016" : {
            "filter" : {
                "term" : {"year" : 2016 }
            }
        }
    }
}
```
## 删除别名
### 1. 删除索引的别名
```
/{index}/_alias/{name}
```
* index：* | _all | glob pattern | name1, name2, …

* name： * | _all | glob pattern | name1, name2, …
## 查询别名

```
GET /logs_20162801/_alias/*
```
## 写索引
一个别名只能有一个写索引，向别名写入的时候就会写入这个索引。当别名指向多个写索引，则写入操作被拒绝。
### 1. 创建别名的时候指定写索引，只能在只有一个索引的时候。

```
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias1",
                 "is_write_index" : true
            }
        }
    ]
}
```
### 3. 创建索引的时候指定写索引

```
PUT my_logs_index-000001
{
  "aliases": {
    "logs": { "is_write_index": true } 
  }
}
```

### 2. 更改写索引

```
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias1",
                 "is_write_index" : true
            }
        }, {
            "add" : {
                 "index" : "test2",
                 "alias" : "alias1",
                 "is_write_index" : false
            }
        }
    ]
}
```