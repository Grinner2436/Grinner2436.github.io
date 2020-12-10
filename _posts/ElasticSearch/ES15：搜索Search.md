---
layout: post
title: 搜索
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 搜索
---
# ES15：搜索
### 分页 From / Size
from : 默认0
size ： 默认10
from + size 不能大于`index.max_result_window`的值，默认是10_000。超过的情况使用滚动搜索API

### 排序Sort 
（1）排序类型

```
GET /my_index/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
* _score : 按分数排序
* _doc : 效率最高的排序，没啥用

（2）多值
可以对多值字段进行函数聚合：
 `min` ， `max`， `sum` ，`avg` ，`median` 
```
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```
（3）嵌套
```
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          <!-指明排序字段->
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             <!-指明嵌套路径和过滤条件->
             "nested": {
                "path": "offer",
                "filter": {
                   "term" : { "offer.color" : "blue" }
                }
             }
          }
       }
    ]
}
```
（4）缺失排序值
```
GET /_search
{
    "sort" : [
        { "price" : {"missing" : "_last"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```
* _last : 被排到最后
* _first : 被排到第一个
* 8.0 / “df” :自定义一个值

（5）忽略缺失字段
unmapped_type 指明了，当一部分索引有此字段，另一部分没有的时候，应该视为大家都有这个字段，这个字段应该是什么类型。
```
GET /_search
{
    "sort" : [
        { "price" : {"unmapped_type" : "long"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```
（6）脚本排序


