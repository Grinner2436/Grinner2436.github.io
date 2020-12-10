---
layout: post
title:  索引模板
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note:  索引模板
---
# 索引模板


#### 创建
```
PUT _template/template_1
{
  "index_patterns": ["ck_t_*", "ck_s_*"],
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_doc": {
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}

```
#### 删除

```
DELETE  /_template/template_1

```

#### 查询 
```
#ID
GET  /_template/template_1

#匹配
GET  /_template/temp*  

#多值
GET  /_template/template_1,template_2

#列表
GET  /_template
```
#### 存在性

```
HEAD _template/template_1
```

