---
layout: post
title: 搜索API
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 搜索API
---
# ES15：搜索API
### 计数
计数API用于统计符合条件的结果数，无需返回文档。用法和_search一致。
```
GET /twitter/_doc/_count
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
### 校验
校验API用于检测搜索条件是否能成功执行。
```
GET twitter/_doc/_validate/query
{
  "query": {
    "query_string": {
      "query": "post_date:foo",
      "lenient": false
    }
  }
}

{"valid":false,"_shards":{"total":1,"successful":1,"failed":0}}
```
### 版本
返回每个文档的版本信息
```
GET /_search
{
    "version": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
### 字段可用性
```
# 所有索引
GET _field_caps?fields=rating

# 指定索引
GET twitter/_field_caps?fields=rating

# 请求体方式
POST _field_caps
{
   "fields" : ["rating"]
}


# 结果示例
{
    "fields": {
        "rating": { 
            "long": {
                "searchable": true,
                "aggregatable": false,
                "indices": ["index1", "index2"],
                "non_aggregatable_indices": ["index1"] 
            },
            "keyword": {
                "searchable": false,
                "aggregatable": true,
                "indices": ["index3", "index4"],
                "non_searchable_indices": ["index4"] 
            }
        },
        "title": { 
            "text": {
                "searchable": true,
                "aggregatable": false

            }
        }
    }
}
```
| `searchable`               | 是否能被搜索       |
| ---                        | ---               |
| `aggregatable`             | 是否能被聚合       |
| `indices`                  | 字段类型相同的索引  |
| `non_searchable_indices`   | 不可搜索的索引     |
| `non_aggregatable_indices` | 不可聚合的索引     |
