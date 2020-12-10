---
layout: post
title: 空值
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 空值
---
# 空值

### 1.空值类型
普通字段：null
数组：[]
         [null]
### 2.默认行为
空值不能被索引、搜索。

### 3.空值替代
字段定义时，null_value属性被用来定义当字段为空值时，替换成的值。
值必须与本字段的定义的类型相兼容。
```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "status_code": {
          "type":       "keyword",
          "null_value": "NULL" 
        }
      }
    }
  }
}
```


