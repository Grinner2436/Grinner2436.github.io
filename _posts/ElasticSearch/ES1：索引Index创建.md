---
layout: post
title: 索引创建
categories:
  - ElasticSearch
tags:
  - 《ElasticSearch》
note: 索引创建
---
# 索引创建

### 一、创建索引

```
PUT  twitter
```
#### 1. 索引命名限制

*   全小写
*   禁用字符字符： `\`, `/`, `*`, `?`, `"`, `<`, `>`, `|`, ` ` (space character), `,`, `#`
*   7.0以前可用 (`:`), 之后的版本禁止
*   不可用在开头：`-`, `_`, `+`
*   不可以叫做：`.` or `..`
*   不能长于255 字节， (注意是字节，所以多字节字符会快速用完字节数，比如汉字)

#### 2. 请求体功能
[配置索引](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/indices-create-index.html#create-index-settings)
[配置映射](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/indices-create-index.html#mappings)
[配置别名](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/indices-create-index.html#create-index-aliases)

#### 3. 请求结果
[结果功能与配置](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/indices-create-index.html#create-index-wait-for-active-shards)

### 二、索引文档时自动创建索引

索引文档时，如果索引不存在，将会被自动创建。

### 1. 配置开关
   通过配置控制是否开启自动创建索引
```
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}
+:允许匹配的索引被自动创建
-:禁止匹配的索引被自动创建

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" 
    }
}
false:禁止所有的索引被自动创建
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
true:允许所有的索引被自动创建
```
### 三、索引模板
（1）索引模板通过一个表达式，匹配到一系列索引名。当匹配的索引被创建时，模板里的settings和mappings将会被应用。

（2）创建索引时，请求体内的settings和mappings将覆盖模板内的设置。

（3）索引模板既会在Create Index时触发，也会在自动创建索引时触发。并且前者因为可以带请求体，所以可以应用（2）的条件。后者因为无法带请求体，将会完全应用模板内的设置。

（4）多个模板匹配同一索引，将会自动合并配置项，高序号（order）的会覆盖低序号的配置。总体顺序：
请求体 > order = 2 > order = 1

（5）模板内支持 C-style/* */的注释。
```
PUT _template/template_1
{
	"index_patterns": ["te*", "bar*"],
	"settings": {
		"number_of_shards": 1
	},
	"order"  :  1,
	"mappings": {
		"_doc": {
			"_source": {
				"enabled": false
			},
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
	},
	"aliases": {
		"alias1": {},
		"alias2": {
			"filter": {
				"term": {
					"user": "kimchy"
				}
			},
			"routing": "kimchy"
		},
		"{index}-alias": {}
		/*{index}占位符将会被替换为真的索引名*/
	}
}
```



