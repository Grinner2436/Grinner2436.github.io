---
layout: post
title: 文档Doc创建
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 文档Doc创建
---
# 文档Doc创建
### 一、索引文档
#### 1. 指定ID索引文档
index api 能够插入或者更新一个文档。如果索引不存在，将会自动创建此索引，并应用匹配的索引模板。详见ES1索引Index创建。

```
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```
这里的更新，并不是增量更新字段，而是数据全覆盖，版本发生改变。

#### 2.版本Version
索引文档的时候，会返回一个版本字段。

```
{
  "_index": "ck_test",
  "_type": "_doc",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}
```
索引文档的时候，可以带上version字段。并遵循version_type规则:

* internal （默认。1 - 无穷）：提供的version和已存在的version必须一致
* external / external_gt : 提供的version必须大于已存在的，或者不存在已有的version
* external_gte :提供的version必须大于等于已存在的version，不建议使用

#### 3.操作类型
默认情况下，存在的id会被视为更新，可以通过op_type参数来设置为仅创建（put-if-absent）。

```
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```
#### 4.不指定ID索引文档
op_type自动被设置为create
```
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

