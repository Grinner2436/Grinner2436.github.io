---
layout: post
title: 文档更新
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 文档更新
---
# ES13：文档更新
## 更新文档
### 1. UPDATE 更新文档
（1）逻辑型更新是通过脚本实现的。
```
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```
（2） 文档型更新是通过提供部分文档实现的。
新文档和源文档会合并。
```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```
（3）如果脚本和文档一起提供，那么文档会被忽略，脚本会被应用。

### 2. UPSERT更新文档
（1）默认行为
如果文档不存在，则使用upsert插入文档。
如果文档已存在，则使用script更新文档。
```
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```
（2） scripted_upsert
使用脚本初始化文档
```
POST sessions/session/dh3sgudg8gsrgl/_update
{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}
```
（3） doc_as_upsert

```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
```
### 3.更新API参数
* retry_on_conflict ：更新失败后，抛出异常前的重试次数。

## 条件更新
### 1. 基本使用

```
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

### 2. 数据吸取
在关闭动态映射Dynamic Mapping的情况下，向索引添加不存在的字段Field，只会存储字段和值，而不会将其索引，即不可搜索。

后面通过Put Mapping 向索引添加字段，匹配的值仍然不可以搜索。

可以通过无指向的update by query，促使所有已索引的 字段把已存在的数据吸取到索引内。

```
POST test/_update_by_query?refresh&conflicts=proceed
```
### 3. 任务控制
update by query任务比较久，可以通过任务接口控制。
```
# 通过TaskAPI可以查看运行中的任务
GET _tasks?detailed=true&actions=*byquery
# 取消任务
POST  _tasks/r1A2WoRbTwKZ516z6NEs5A:36619/_cancel
# 控制速度
POST  _update_by_query/r1A2WoRbTwKZ516z6NEs5A:36619/_rethrottle?requests_per_second=-1
```


