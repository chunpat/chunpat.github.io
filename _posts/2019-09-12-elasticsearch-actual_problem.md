---
layout: post
title: elasticsearch业务问题
date: 2019-09-12 
tags: elasticsearch
---

## 场景
> 要搜索的内容

```
P92325 iphone 7s plus 4G 黑色 美版 无锁
P92324 iphone 7 128g 金色 有锁 通用版
```

> 以为客户那边要求是，想的很简单

* 1、iphone7splus  搜索->P92325 iphone 7s plus 4G 黑色 美版 无锁
* 2、iphone 7s plus  搜索->P92325 iphone 7s plus 4G 黑色 美版 无锁


> 实际输入情况,五花八门，还有这些情况

* 1、iphone7splus  搜索->P92325 iphone 7s plus 4G 黑色 美版 无锁
* 2、iphone 7s plus   搜索->P92325 iphone 7s plus 4G 黑色 美版 无锁
* 3、iphone7sp 美版   搜索->P92325 iphone 7s plus 4G 黑色 美版 无锁
* 4、iphone7 128g 美版   搜索->P92324 iphone 7 128g 金色 有锁 通用版
* 5、iphone7s plus 美版   搜索->iphone 7s plus 4G 黑色 美版 无锁
* 。。。

## 解决路程
> 错误解决方案，将搜索内容去掉空格，然后DOC的文章分析用IK分词器ik_max_word，
搜索用ik_smart,然后将搜索的词也去掉空格。

### 1、创建索引
```
curl -XPUT http://localhost:9200/model
```

### 2、设置索引属性map
```
curl -XPOST http://localhost:9200/model/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "name": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            }
        }

}'
```

### 3、写入doc数据
```
curl -XPOST http://localhost:9200/model/_create/1 -H 'Content-Type:application/json' -d'
{"content":"P92325 iphone7splus4G黑色美版无锁"}
'
curl -XPOST http://localhost:9200/model/_create/2 -H 'Content-Type:application/json' -d'
{"content":"P92324 iphone7128g金色有锁通用版"}
'

```
### 4、查看doc状态
```
curl -X GET "http://localhost:9200/_cat/indices?v"
```

### 5、搜索数据

iphone7splus 或 iphone 7s plus 查找搜索出 P92325 iphone 7s plus 4G 黑色 美版 无锁
```
curl -XPOST http://localhost:9200/model/_search  -H 'Content-Type:application/json' -d'
{
    "size" : 100,
    "query" : { "match" : { "content" : "iphone7splus" }}
  
}
'
```
结果，啥都没：
```
{
	"took": 249,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 0,
			"relation": "eq"
		},
		"max_score": null,
		"hits": []
	}
}
```

> 为什么搜不出（一），因为P92325 iphone7splus4G黑色美版无锁 的文章分析不会出现iphone7splus。

搜索词ik_smart分析
```
curl -XGET "http://localhost:9200/model/_analyze" -H 'Content-Type: application/json' -d'
{
   "text":"iphone7splus","tokenizer": "ik_smart"
}'

```

结果出现
```
{
	"tokens": [{
		"token": "iphone7splus",
		"start_offset": 0,
		"end_offset": 12,
		"type": "LETTER",
		"position": 0
	}]
}
```

查找文ik_max_word分析
```
curl -XGET "http://localhost:9200/model/_analyze" -H 'Content-Type: application/json' -d'
{
   "text":"iphone7splus4G黑色美版无锁","tokenizer": "ik_max_word"
}'
```

结果，不会出现iphone7splus
```

{
    "tokens": [
        {
            "token": "iphone7splus4g",
            "start_offset": 0,
            "end_offset": 14,
            "type": "LETTER",
            "position": 0
        },
        {
            "token": "iphone",
            "start_offset": 0,
            "end_offset": 6,
            "type": "ENGLISH",
            "position": 1
        },
        {
            "token": "7",
            "start_offset": 6,
            "end_offset": 7,
            "type": "ARABIC",
            "position": 2
        },
        {
            "token": "splus",
            "start_offset": 7,
            "end_offset": 12,
            "type": "ENGLISH",
            "position": 3
        },
        {
            "token": "4",
            "start_offset": 12,
            "end_offset": 13,
            "type": "ARABIC",
            "position": 4
        },
        {
            "token": "g",
            "start_offset": 13,
            "end_offset": 14,
            "type": "ENGLISH",
            "position": 5
        },
        {
            "token": "黑色",
            "start_offset": 14,
            "end_offset": 16,
            "type": "CN_WORD",
            "position": 6
        },
        {
            "token": "美版",
            "start_offset": 16,
            "end_offset": 18,
            "type": "CN_WORD",
            "position": 7
        },
        {
            "token": "无",
            "start_offset": 18,
            "end_offset": 19,
            "type": "CN_CHAR",
            "position": 8
        },
        {
            "token": "锁",
            "start_offset": 19,
            "end_offset": 20,
            "type": "CN_CHAR",
            "position": 9
        }
    ]
}
```

所以说，去掉空格方案是不行的，那就不去掉咯

分析语句
```
curl -XGET "http://localhost:9200/model/_analyze" -H 'Content-Type: application/json' -d'
{
   "text":"iphone 7s plus 4G 黑色 美版 无锁","tokenizer": "ik_max_word"
}'
```

结果，还是没有iphone7splus组合，那么搜索哪里也不去掉空格吧。这样就可以搜索到了。
```
{
    "tokens": [
        {
            "token": "iphone",
            "start_offset": 0,
            "end_offset": 6,
            "type": "ENGLISH",
            "position": 0
        },
        {
            "token": "7s",
            "start_offset": 7,
            "end_offset": 9,
            "type": "LETTER",
            "position": 1
        },
        {
            "token": "7",
            "start_offset": 7,
            "end_offset": 8,
            "type": "ARABIC",
            "position": 2
        },
        {
            "token": "s",
            "start_offset": 8,
            "end_offset": 9,
            "type": "ENGLISH",
            "position": 3
        },
        {
            "token": "plus",
            "start_offset": 10,
            "end_offset": 14,
            "type": "ENGLISH",
            "position": 4
        },
        {
            "token": "4g",
            "start_offset": 15,
            "end_offset": 17,
            "type": "LETTER",
            "position": 5
        },
        {
            "token": "4",
            "start_offset": 15,
            "end_offset": 16,
            "type": "ARABIC",
            "position": 6
        },
        {
            "token": "g",
            "start_offset": 16,
            "end_offset": 17,
            "type": "ENGLISH",
            "position": 7
        },
        {
            "token": "黑色",
            "start_offset": 18,
            "end_offset": 20,
            "type": "CN_WORD",
            "position": 8
        },
        {
            "token": "美版",
            "start_offset": 21,
            "end_offset": 23,
            "type": "CN_WORD",
            "position": 9
        },
        {
            "token": "无",
            "start_offset": 24,
            "end_offset": 25,
            "type": "CN_CHAR",
            "position": 10
        },
        {
            "token": "锁",
            "start_offset": 25,
            "end_offset": 26,
            "type": "CN_CHAR",
            "position": 11
        }
    ]
}

```

> 但是问题是，这只是解决了我的简单设想客户是这么简单的输入搜索。但是实际却不是，复杂的多。

解决方案是将搜索分析都改为ik_max_word，这样搜索词和文章的颗粒度都分的很细。
```
curl -XPOST http://localhost:9200/model/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "name": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }

}'
```

* 注意的是，已经写入doc数据是不能修改的哦！


## 后记
1、去掉空格这种方案是听老大说的，太过信任和自己的懒惰，没实际去测试，导致进坑多时。

2、探究技术要优先多看官方手册，其他来源资料只会让自己乱上加乱。

3、功能流程要测一遍，很重要，处处都可能留下bug。错了之后数据乱，就更难调了。


## 参考
* 1、[GitHub - medcl/elasticsearch-analysis-ik: The IK Analysis plugin integrates Lucene IK analyzer into elasticsearch, support customized dictionary.](https://github.com/medcl/elasticsearch-analysis-ik)
