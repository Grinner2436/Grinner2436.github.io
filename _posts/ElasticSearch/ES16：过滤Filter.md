---
layout: post
title: 过滤
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 过滤
---
# ES16：过滤

## 后置过滤器post_filter
后置过滤器使得 “查询 +  过滤 + 聚合” 可以拆分为：
查询 + 过滤
查询 + 聚合 
查询 + 过滤 + 聚合

例如：

```
PUT /shirts/_doc/1?refresh
{
    "brand": "gucci",
    "color": "red",
    "model": "slim"
}
```

（1）你希望把红色的挑出来作为本次查询结果，但是又提供所有颜色的分类。也就是：

```
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } 1
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } 2
    },
    "color_red": {3
      "filter": {
        "term": { "color": "red" } 
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } 
        }
      }
    }
  },
  "post_filter": { 4
    "term": { "color": "red" }
  }
}
```
1 + 4 = 查询 + 过滤
1 + 2 = 查询 + 聚合
1 + 3 = 查询 + 过滤 + 聚合
（2）原理
在查询条件放宽，把过滤下放到聚合3上，这样聚合2的查询就不会被过滤。
聚合完毕之后再通过后置过滤器4把结果限制一下。这样，主聚合3就和查询结果的过滤条件一致了。
注意区分主聚合。


