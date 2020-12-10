---
layout: post
title: Mapping创建
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: Mapping创建
---
# Mapping创建

### Type解决方案
* 使用自定义的一个type字段，取代_type字段

### 字段映射属性
映射属性，指创建索引时，mappings._doc下面的直接属性。

```
PUT my_index
{
  "mappings": {
    "_doc": {
      "dynamic": false
    }
  }
}
```
1.`dynamic`
       true ：开启自动映射，未知字段自动附加到Mapping
       false : 关闭自动映射，未知字段被忽略，不能被搜索，但是在source里会包含。
       strict : 禁止添加新字段，未知字段将被拒绝并返回异常。


### 动态映射
| **JSON datatype**     | **Elasticsearch datatype**                                   |
| --------------------- | ------------------------------------------------------------ |
| `null`                | 不添加字段                                           |
| `true`                | `boolean` |
| 浮点数                 | `float`|
| integer               | `long` |
| object                | `object`|
| array                 | 取决于数组内第一个非空值|
| string[`date`]        | `date`|
| string[`numeric`]     | `double`|
| string[`numeric`]     | `long`|
| string                | `text` + `keyword`

#### 1. 日期配置

```
PUT my_index
{
  "mappings": {
    "_doc": {
      /*默认是true*/
      "date_detection": false,
      /*默认是yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z*/
      "dynamic_date_formats":  ["MM/dd/yyyy"]
    }
  }
}
```
#### 2.数字配置

```
PUT my_index
{
  "mappings": {
    "_doc": {
      /*默认是false*/
      "numeric_detection": true
    }
  }
}
```
### 动态模板
动态映射模板用于自定义一些特殊转换。

```
# 将名称以long_开头的，不是以_text结尾的所有string类型的原值，映射为long值
PUT my_index
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        {
          "longs_as_strings": {
            "match_mapping_type": "string",
            "match":   "long_*",
            "unmatch": "*_text",
            "mapping": {
              "type": "long"
            }
          }
        }
      ]
    }
  }
}

#使用两个占位符获取上下文变量
{name}：字段名称
{dynamic_type}：推断的类型

PUT my_index
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        {
          "named_analyzers": {
            "match_mapping_type": "string",
            "match": "*",
            "mapping": {
              "type": "text",
              "analyzer": "{name}"
            }
          }
        },
        {
          "no_doc_values": {
            "match_mapping_type":"*",
            "mapping": {
              "type": "{dynamic_type}",
              "doc_values": false
            }
          }
        }
      ]
    }
  }
}
```



