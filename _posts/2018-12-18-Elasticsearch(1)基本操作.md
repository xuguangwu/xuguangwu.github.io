---
title: Elasticsearch(1)基本操作
categories:
 - Java
tags:
 - Java
---

本地运行elasticsearch服务和kibana服务，
在kibana中用dev tools直接操作es。

学习使用API

* GET _cat/health?v
* GET _cat/indices?v
* PUT /test_index
* DELETE /test_index
* DELETE /ecommerce

# 添加index
PUT /ecommerce/_create/1
{
    "name" : "gaolujie yagao",
    "desc" :  "gaoxiao meibai",
    "price" :  30,
    "producer" :      "gaolujie producer",
    "tags": [ "meibai", "fangzhu" ]
}

PUT /ecommerce/_create/2
{
    "name" : "jiajieshi yagao",
    "desc" :  "youxiao fangzhu",
    "price" :  25,
    "producer" :      "jiajieshi producer",
    "tags": [ "fangzhu" ]
}

PUT /ecommerce/_create/3
{
    "name" : "zhonghua yagao",
    "desc" :  "caoben zhiwu",
    "price" :  40,
    "producer" :      "zhonghua producer",
    "tags": [ "qingxin" ]
}

PUT /ecommerce/_create/4
{
    "name" : "special yagao",
    "desc" :  "caoben zhiwu",
    "price" :  40,
    "producer" :      "special yagao producer",
    "tags": [ "special" ]
}


# query string search
GET /ecommerce/_search?q=name:yagao&sort=price:desc
# query DSL
GET /ecommerce/_search
{
  "query": {
    "match": {
      "name": "yagao"
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}

# 分页查询
GET /ecommerce/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}

# 展示字段
GET /ecommerce/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["name","price"]
}

# 添加过滤查询
GET /ecommerce/_search
{
  "query": {
    "bool": {
    "must": [
      {"match": {
        "name": "yagao"
      }}
    ]
    , "filter": {"range": {
      "price": {
        "gt": 25,
        "lte": 40
      }
    }}
    }
  }
}

# 全文检索 
GET /ecommerce/_search
{
  "query": {
    "match": {
      "producer": "yagao producer"
    }
  }
}

# 短语搜索
GET /ecommerce/_search
{
  "query": {
    "match_phrase": {
      "producer": "yagao producer"
    }
  }
}

# 高亮搜索结果
GET /ecommerce/_search
{
  "query": {
    "match": {
      "producer": "yagao producer"
    }
  },
  "highlight": {
    "fields": {
      "producer":{}
    }
  }
}






