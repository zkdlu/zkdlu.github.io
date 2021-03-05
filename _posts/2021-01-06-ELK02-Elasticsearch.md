---
layout: post
title: "[ELK] 2. Elasticsearch"
description: "Elasticsearch란?"
date: 2021-01-06 00:00:01
tags: [elk]
comments: true
share: true
---

# Elasticsearch란

Apache Lucene 기반으로 개발 된 실시간 분산 검색 및 분석 엔진



## RDBMS와의 차이

| RDBMS              | ElasticSearch         |
| ------------------ | --------------------- |
| Database           | Index                 |
| Table              | Type                  |
| Row                | Document              |
| Column             | Field                 |
| Index              | Analyze               |
| Primary key        | _id                   |
| Schema             | Mapping               |
| Physical partition | Shard                 |
| Logical partition  | Route                 |
| Relational         | Parent/Chille, Nested |
| SQL                | Query DSL             |



### 도커로 설치해보기

```bash
$ docker pull docker.elastic.co/elasticsearch/elasticsearch:7.6.2

$ docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.6.2
```

> 9200: http 포트,  9300: 전송포트



- **생성** - /{index}/{type}/{id}

  > id를 입력하지 않으면 랜덤한 문자열을 id로 생성한다.

  Request

  ```bash
  POST http://localhost:9200/database/user/1
  {
    "name": "zkdlu",
    "age": 25
  }
  ```

  Response

  ```json
  {
    "_index": "database",
    "_type": "user",
    "_id": "5",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 5,
    "_primary_term": 1
  }
  ```



- **수정** - /{index}/{type{id}}

Request

  ```bash
  PUT http://localhost:9200/database/user/1
  {
    "name": "zkdlu",
    "age": 24
  }
  ```

  Response

  ```json
  {
     "_index": "database",
     "_type": "user",
     "_id": "1",
     "_version": 3,
     "result": "updated",
     "_shards": {
         "total": 2,
         "successful": 1,
         "failed": 0
     },
     "_seq_no": 6,
     "_primary_term": 1
  }
  ```
  
  
  
- **조회** - /{index}/{type}/{id}
 
Request

  ```bash
  GET http://localhost:9200/database/user/1
  ```
  
  Response
  
  ```json
{
      "_index": "database",
      "_type": "user",
      "_id": "1",
      "_version": 3,
      "_seq_no": 6,
      "_primary_term": 1,
      "found": true,
      "_source": {
          "name": "zkdlu",
          "age": 24
      }
  }
  ```
  
  
  
- **삭제** - /{index}/{type}/{id}

Request

  ```bash
  DELETE http://localhost:9200/database/user/1
  ```
  
  Response
  
  ```json
{
      "_index": "database",
      "_type": "user",
      "_id": "1",
      "_version": 4,
      "result": "deleted",
      "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
      },
      "_seq_no": 7,
      "_primary_term": 1
  }
  ```



- **전체 조회** - /{index}/{type}/_search

Request

  ```bash
  GET http://localhost:9200/database/user/_search
  ```
  
  Response
  
  ```json
{
      "took": 945,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 4,
              "relation": "eq"
          },
          "max_score": 1.0,
        "hits": [
             {
                 "_index": "database",
                 "_type": "user",
                 "_id": "2",
                 "_score": 1.0,
                 "_source": {
                     "name": "zkdlu2",
                     "age": 25
                 }
             },
             ... 생략
  ```




- **전체 조건 조회** - /{index}/{type}/_search?q={key}:{value}

Request

  ```bash
  GET localhost:9200/database/user/_search?q=name:zkdlu
  ```
  
  Response
  
  ```json
{
      "took": 2,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 1,
              "relation": "eq"
          },
          "max_score": 1.0296195,
          "hits": [
              {
                  "_index": "database",
                  "_type": "user",
                  "_id": "1",
                  "_score": 1.0296195,
                  "_source": {
                      "name": "zkdlu",
                      "age": 25
                  }
              }
          ]
      }
  }
  ```



## 버전 업그레이드

작성 후 알았는데 위의 API는 Elasticsearch 6.x 버전에서 사용 되는 api이다.

7.x 로 올라오면서 Type 구조를 삭제하고 Document는 _doc으로 대체되었습니다.



### 변경 된 API 비교

- **Search**

  | API              | 6.x                               | 7.x                        |
  | ---------------- | --------------------------------- | -------------------------- |
  | search           | /{index}/{type}/_search           | {index}/_search            |
  | msearch          | /{index}/{type}/_msearch          | /{index}/_msearch          |
  | count            | /{index}/{type}/_count            | /{index}/_count            |
  | explain          | /{index}/{type}/{id}/_explain     | /{index}/_explain/{id}     |
  | search template  | /{index}/{type}/_search/template  | /{index}/_search/template  |
  | msearch template | /{index}/{type}/_msearch/template | /{index}/_msearch/template |



- **Document**

  | API          | 6.x                              | 7.x                       |
  | ------------ | -------------------------------- | ------------------------- |
  | index        | {index}/{type}/{id}              | /{index}/**_doc**/{id}    |
  | delete       | /{index}/{type}/{id}             | /{index}/**_doc**/{id}    |
  | get          | /{index}/{type}/{id}             | /{index}/**_doc**/{id}    |
  | update       | /{index}/{type}/{id}/_update     | /{index}/_update/{id}     |
  | get source   | /{index}/{type}/{id}/_source     | /{index}/_source/{id}     |
  | bulk         | /{index}/{type}/_bulk            | /{index}/_bulk            |
  | mget         | /{index}/{type}/_mget            | /{index}/_mget            |
  | termvectors  | /{index}/{type}/{id}/_termvector | /{index}/_termvector/{id} |
  | mtermvectors | /{index}/{type}/_mtermvectors    | /{index}/_mtermvectors    |

  

- **Index**  
  
  | API               | 6.x                                     | 7.x                              |
  | ----------------- | --------------------------------------- | -------------------------------- |
  | create index      | /{index}                                | ㅡ                               |
  | get mapping       | /{index}/_mapping/{type}                | /{index}/_mapping                |
  | put mapping       | /{index}/_mapping/{type}                | /{index}/_mapping                |
  | get field mapping | /{index}/{type}/_mapping/field/{fields} | /{index}/_mapping/field/{fields} |
  | get template      | /_template/{template}                   | ㅡ                               |
  | put template      | /_template/{template}                   | ㅡ     |
