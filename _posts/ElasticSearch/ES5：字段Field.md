---
layout: post
title: 字段Field
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 字段Field
---
# 字段Field

## String字符串类型

### 1.text
会被分词，不能被聚集、排序。

### 2. keyword
只能通过全词匹配搜索，不能通过模糊匹配搜索。

## 数字类型

### 3. 整型 （`byte`, `short`, `integer`，`long`)
（1）尽量选择符合要求的更小数字类型。能加快索引和搜索的速度。
（2）无论选择哪种类型，储存所费的空间都只和实际值有关。
### 4. 浮点型（`scaled_float`,`double`, `float`，`half_float`）
（1）把浮点型通过倍数扩充，变成整型存储，会节约存储空间。因为整型容易压缩。而实现这一目的可以用scaled_float类型。

```
fractions ：乘以的倍数，得出乘积以后，小数部分被舍弃。
2.34 * 10  = 23（保存） = 2.3(使用)
```

（2）当scaled_float无法满足，如小数位过多的时候。应尽量选择符合要求的更小数字类型。
| Type | Minimum value | Maximum value | Significant bits / digits |
| :-- | :-- | :-- | :-- |
| `double` |`2-1074`|`(2-2-52)·21023`|`53` / `15.95`|
|     |     |     |     |
| `float` | `2-149` | `(2-2-23)·2127` | `24` / `7.22` | 
|     |     |     |     |
| `half_float` | `2-24` | `65504` | `11` / `3.31` |

## 日期类型
### 5. date
（1）内部存储为毫秒，date类型的查询，都会转换为range查询。
（2）定义的时候，日期可以指定格式，第一种格式会被用作查询返回值的格式化。

```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}
```
## 布尔类型
### 6. boolean
（1）往boolean上赋值时，既可以赋值true，false，也可以赋值“true”，“false”
（2）聚集函数如terms aggregation,把1 和 0 当做key，把字符串“true”和“false”当做key_as_string。
（3）当作为脚本返回值时，boolean类型返回 1 或者 0。

```
# 文档
{
  "user": true,
  "man": "true"
}

#查询
GET ck_test/_search
{
  "aggs": {
    "user": {
      "terms": {
        "field": "user"
      }
    }
  },
  "script_fields": {
    "user": {
      "script": {
        "lang": "painless",
        "source": "doc['user'].value"
      }
    }
  }
}

#结果
{
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "ck_test",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "fields": {
          "user": [
            true
          ]
        }
      }
    ]
  },
  "aggregations": {
    "user": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          /*聚集函数如terms aggregation,把1 和 0 当做key*/
          "key": 1, 
          /*把字符串“true”和“false”当做key_as_string*/
          "key_as_string": "true"          
        }
      ]
    }
  }
}
```
## 二进制类型
### 7. binary
binary类型用于存储base64编码的字符串，不能搜索也不能排序。

```
在网络上交换数据时，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，所以就先把数据先做一个Base64编码，统统变成可见字符，这样出错的可能性就大降低了。

任何一个数据无非可以看作一个比特流，如01000100010011101100111010111100011001010......那么我们取6个比特为一组，计算它的ascii值，得到一个字符，这个字符肯定是可见字符，好，把它对应的字符写出来，再取6个比特，计算...，如此下去，直到最后，就完成了编码。
```
## 范围类型
### 索引和查询
（1）范围索引使用上下端来标识
```
PUT range_index/_doc/1?refresh
{
  "expected_attendees" : { 
    "gte" : 10,
    "lte" : 20
  },
  "time_frame" : { 
    "gte" : "2015-10-31 12:00:00", 
    "lte" : "2015-11-01"
  }
}
```

（2）范围查询支持一个关系限定：

```
GET range_index/_search
{
  "query" : {
    "range" : {
      "time_frame" : { 
        "gte" : "2015-10-31",
        "lte" : "2015-11-01",
        "relation" : "within" 
      }
    }
  }
}
```
`relation` parameter which can be one of `WITHIN`, `CONTAINS`, `INTERSECTS` (default)

### 8. 整型范围（`integer_range`,`long_range`）
### 9. 浮点型范围（`float_range`,`double_range`）
### 10. 日期范围（`date`）
（1）定义字段时，可选参数和[date](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/date.html )格式一致。
（2）索引文档时，文档格式和[date range queries](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-range-query.html#ranges-on-dates "Ranges on date fields")格式一致
### 11. IP范围（`ip_range`）
IP范围在索引的时候可以用CIDR格式

## 复合类型
### 12. 对象类型 （`object`）
（1）任意深度嵌套的json在es内部都默认存储为object类型，但无需在映射中指明。
（2）任意深度的json在es内部都只会扁平化为普通文档，嵌套字段用`.`分隔不同深度的键。
（2）array的扁平化，是把多个对象的相同属性，扁平为一个多值列表。

```
PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
# 被ES扁平化之后
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```
### 13. 嵌套类型 （`nested`）
（1）nested是object的特殊化版本，允许保存json array。并且在查询时，array内的值互相独立，每个值都会单独存储为一个隐藏文档。
（2）嵌套文档可以被
*  使用 `Nested Query`查询
*  使用`Nested· Aggregation`和`reverse_nested` 聚集函数分析
*  使用 `nested sorting`排序
*  使用 `nested inner hits`检索和高亮
（3）嵌套文档的限制
有100个嵌套字段的文档，实际上会被存储为101个文档。为了防止映射激增，默认的每个索引的嵌套字段个数已经被限制为50个。

### 14. array类型，一个字段多个值
（1）es并没有显示地提供array类型，但是每种字段类型都可以存储多个值，并且会被自动转换为array格式。
（2）特征
* 将一组值索引到一个尚未存在的字段时，es根据第一个值确定字段的类型，然后将剩余值强转为第一个值的类型，无法强转会报错。
* 空数组[]会被视为此字段没有值。
`An empty array [] is treated as a missing field — a field with no values.`
* 数组可以包含 `null`,要么会被跳过，要么会被配置的值取代。

### 15.field属性，一个字段多个索引
```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "text": { 
          "type": "text",
          "fields": {
            "english": { 
              "type":     "text",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}
```
## 功能性类型
### 16. ip
ip类型用于存储ip地址
### 17. completion
用于自动完成功能
### 18. token_count
接收String，使用分词器分词，计算分出的词汇数量。

```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "name": { 
          "type": "text",
          "fields": {
            "length": { 
              "type":     "token_count",
              "analyzer": "standard"
            }
          }
        }
      }
    }
  }
}
```
### 19. percolator（渗滤器）
在查询的时候，提供文档，然后会搜索出匹配文档的渗滤条件。

一般的查询，是提供查询条件，查出符合条件的文档。文档内容存储在字段值里，查询语句写在查询条件里。
渗滤器恰恰相反，它存储的字段值是查询语句，查询条件是文档内容。

它的作用是查询文档所符合的渗滤条件。

主要可以用于内容分类。

```
# 定义一个渗滤器,功能是给文档分类
PUT index
{
  "mappings": {
    "_doc" : {
      "properties": {
        "query" : {
          "type" : "percolator"
        },
        "doc_type" : {
          "type": "text"
        }
      }
    }
  }
}
# 定义其中一种分类，如果文档的body字段含有“人”字，则文档分类为human
PUT queries/_doc/1?refresh
{
  "query" : {
    "match" : {
      "body" : "人"
    }
  },
  "doc_type"  :  "human"
}
```
### 20. alias
（1）字段可以设置为别名类型，用于指向同文档的另一个字段。被指向的字段必须用全路径标识。

```
PUT trips
{
  "mappings": {
    "_doc": {
      "properties": {
        "distance": {
          "type": "long"
        },
        "route_length_miles": {
          "type": "alias",
          "path": "distance" 
        }
      }
    }
  }
}
```
（2）别名目标
别名目标有一定的限制。
* 必须是具体类型，不能是object类型也不能是其他别名。
* 别名创建时，目标字段必须存在
* 如果指向nested类型，别名必须有和目标一样的嵌套范围。
* 别名只能搜索不能写入。



