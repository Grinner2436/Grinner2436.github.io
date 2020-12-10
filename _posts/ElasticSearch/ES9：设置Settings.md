# ES9：设置Settings

## 获取设置

```
# 单个
GET /twitter/_settings
# 多值
GET  /twitter,kimchy/_settings  
# 全部
GET  /_all/_settings  
# 通配
GET  /log_2013_*/_settings
```

## 恢复默认值
设置为null
```
PUT /twitter/_settings
{
    "index" : {
        "refresh_interval" : null
    }
}
```
## 定义新分词器
需要先关闭索引，再打开索引
```
POST /twitter/_close

PUT /twitter/_settings
{
  "analysis" : {
    "analyzer":{
      "content":{
        "type":"custom",
        "tokenizer":"whitespace"
      }
    }
  }
}

POST /twitter/_open
```

