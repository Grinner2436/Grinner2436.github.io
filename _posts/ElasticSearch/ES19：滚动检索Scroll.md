---
layout: post
title: 滚动检索Scroll
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 滚动检索Scroll
---
# ES19：滚动检索Scroll
滚动检索类似于DataBase的Cursor，可以将一次查询的结果集进行滚动浏览，以免结果集太大时，一次取回太多出现问题。

```
# 首次搜索需要指定滚动上下文存留时间
POST /twitter/_search?scroll=1m
{
    "size": 100,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
# 后续搜索需要换一个接口
POST /_search/scroll 
{
    "scroll" : "1m", 
    "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==" 
}
```
参数：
  * scroll ：本次滚动上下文的存留时间，每次滚动查询都增加额外的时间。
  * scroll_id ： 滚动上下文的标识符。每次滚动查询时，这个ID可能会变，所以应该使用最新返回的一个。

### 
  * 滚动查询只有第一次查询支持聚合。
  * 滚动查询对于_doc类型的排序有优化，如果想要滚动所有结果而不需要排序的话，建议使用这种排序。

### 清除滚动上下文

```
DELETE /_search/scroll
{
    "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}

# 删除所有
DELETE  /_search/scroll/_all
```
### `search_after`游标
search after提供一种滚动的新方式替代scroll。
```
GET twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    },
    "search_after": [1463538857, "654323"],
    "sort": [
        {"date": "asc"},
        {"tie_breaker_id": "asc"}
    ]
}
```
参数是之前的 查询所返回的结果的排序字段。

总是基于最新版本的数据来跳过数据。如果在上次搜索之后，有数据新增或者修改，是无法被下次搜索感知的。



