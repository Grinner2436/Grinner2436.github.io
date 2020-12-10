---
layout: post
title: 文档操作
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 文档操作
---
# ES12：文档操作

## GET 查询文档
GET方式查询文档，返回的是实时数据，在返回之前，会调用refresh来确保最新文档可见。
```
GET twitter/_doc/0

# 禁用实时检索
GET  twitter/_doc/0?realtime=false
GET  twitter/_doc/0?refresh=false
# 不返回源文档
GET  twitter/_doc/0?_source=false

# 指定包括的字段
GET  twitter/_doc/0?_source=*.id,retweeted

# 指定包括的和排除的字段
GET  twitter/_doc/0?_source_include=*.id&_source_exclude=entities

# 直接查询源文档
GET twitter/_doc/1/_source
#过滤字段的方式与检索文档一样
```

## MGET 查询多个文档

```
# 按ID查询
GET /test/_doc/_mget
{
    "ids" : ["1", "2"]
}
# 自定义类型
GET /test/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
# 自定义索引、源文档刷选
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
```

## DELETE 删除文档

```
# ID删除
DELETE /twitter/_doc/1
# 条件删除
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
```
## Bulk批量操作

```
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```


