---
layout: post
title: 文档迁移
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 文档迁移
---
# ES14：文档Doc迁移
## 复制文档
### 1.空索引迁移
文档迁移仅仅是把文档从一个索引，复制到另一个索引。
* 需要手动创建目的地索引
* 需要手动设置目的地索引的映射、设置等

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
```
### 2.存在索引迁移
存在的索引，将会像Index操作一样，覆盖目的地的文档。
可以用op_type，控制禁止覆盖，只迁移目的地没有的文档。
重复的ID会导致版本冲突。
```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```
### 3.通过控制参数，达成特殊目的

```
# 将最新的10000条复制出来
POST _reindex
{
  "size": 10000,
  "source": {
    "index": "twitter",
    "sort": { "date": "desc" }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

* size ：限制迁移复制的文档数量
* sort：控制排序方式

## 控制过程
### 1.控制
可以通过脚本控制迁移行为，并且脚本中可以修改元数据。
也可以通过query字段，筛选源索引中的数据。
  *   `_id`
  *   `_type`
  *   `_index`
  *   `_version`
  *   `_routing`
```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  },
  "script": {
    "source": "if (ctx._source.foo == 'bar') {ctx._version++; ctx._source.remove('foo')}",
    "lang": "painless"
  }
}

```
字段重命名
```
POST _reindex
{
  "source": {
    "index": "test"
  },
  "dest": {
    "index": "test2"
  },
  "script": {
    "source": "ctx._source.tag = ctx._source.remove(\"flag\")"
  }
}
```

### 2.相同ID时行为
ctx.op 控制了当目的地和源索引有相同文档时的行为：
  * noop : 忽略源索引的文档
  * delete：删除目的地文档，使用源索引文档
  * 
## 远程索引复制
可以从远程主机复制索引
```
POST _reindex
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
```

