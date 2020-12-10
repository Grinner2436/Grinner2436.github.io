---
layout: post
title: 打分Score
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 打分Score
---
# ES17：打分Score

### explain打分详情
```
GET /_search
{
    "explain": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
### explain打分匹配
打分匹配API用于解释某个文档的匹配分，可以查询出某个文档是否匹配某个查询。
```
GET /twitter/_doc/0/_explain
{
    "query" : {
      "match" : { "message" : "elasticsearch" }
    }
}
```

### 加权
```
GET /_search
{
    "indices_boost" : [
        { "alias1" : 1.4 },
        { "index*" : 1.3 }
    ]
}
```

### 最低评分要求
低于评分要求的文档将会被排除
```
GET /_search
{
    "min_score": 0.5,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```
## 二次评分 rescore
二次评分可以提高排序精确度。实现方式是取一次评分的前N个文档，通过查询query和后置过滤器post_filter进行二次评分。这样的评分方式可以缩小第二次评分的文档范围，减少开销。

目前支持的二次评分器只有一个：查询评分器。

```
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : [ {
      "window_size" : 100,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }, {
      "window_size" : 10,
      "query" : {
         "score_mode": "multiply",
         "rescore_query" : {
            "function_score" : {
               "script_score": {
                  "script": {
                    "source": "Math.log10(doc.likes.value + 2)"
                  }
               }
            }
         }
      }
   } ]
}
```
### 相关重要性

  * query_weight：查询分的系数，默认1
  * query_weight：重排分的系数，默认1
  * 
### 分数叠加模式 score_mode

| Score Mode | Description               |
| ---        | ---                       |
| `total`    | 总和模式，两数相加。默认值。 |
| `multiply` | 两数相乘                   |
| `avg`      | 两数平均                   |
| `max`      | 取最大值                   |
| `min`      | 取最小值                   |

### 多次评分
多个重评分器，依次会以前一个重评分器的结果作为重评分依据。