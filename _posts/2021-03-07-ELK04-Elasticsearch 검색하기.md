---
layout: post
title: "[ELK] 4. Elasticsearch 검색하기"
description: "Elasticsearch를 사용해 다양한 검색을 해보자"
date: 2021-03-07 00:00:01
tags: [elk]
comments: true
share: true
---



# Elasticsearch를 이용해 검색

### 사전준비

Elasticsearch에서 제공하는 대량의 테스트 데이터를 Bulk API를 호출하여 저장해준다.

Data: [data.json](https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json)

```bash
$ docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.10.2
$ curl -XPOST 'localhost:9200/account/_bulk?pretty' -H "Content-Type: application/json" --data-binary "@data.json"
```

후 http://localhost:9200/account/_search 로 접속 해본다.

## 검색 Response

```json
{
    "took": "검색하는데 걸린 시간(ms)",
    "timed_out": "시간 초과 여부 (true/false)",
    "_shards": {
        "total": "검색한 shard 수",
        "successful": "검색에 성공한 shard 수",
        "skipped": 0,
        "failed": "검색에 실패한 shard 수"
    },
    "hits" : {
        "total": "검색 된 문서의 총 개수",
        "max_score": "검색 조건과 일치한 점수의 최대 값",
        "hits" : [ 
            {
                "_index": "인덱스 이름",
                "_type": "_doc", # 7.x부터 _doc으로 통일,
                "_id": "id",
                "_score": "해당 document가 검색 쿼리와 일치하는 상대적인 값",
                "_source": {
                    # 데이터
                }
            }
        ]
    }
}
```




## URI Search

URL에 파라미터를 넘기는 방법

```bash
http://localhost:9200/account/_search?q=gender:F&size=1&from=1
```

> 저장된 계좌정보중에서 성별이 여자인 결과를 1번째부터 1개만 가져오기 (기본은 0번째부터)



## Query DSL

json 파일에 쿼리를 작성하여 넘기는 방법. 더욱 상세한 표현이 가능

- Query Context

  Document가 Query와 얼마나 일치하는가를 _score로 응답

- Filter Context

  Document가 Query와 얼마나 일치하는가를 True/False로 응답



### 사용법

```bash
$ curl -XGET 'localhost:9200/account/_search?pretty' -H 'Content-Type: application/json' -d @query.json
```

> query.json 에 검색 쿼리를 작성한다.




### 전체 검색 match_all

```json
// match_all.json
{
    "query": {
        "match_all": {}
    }
}
```

### 기본 필드 검색 match

```json
// match.json
{
    "query": {
        "match": {
            "gender": "F"
        }
    }
}
```

### True/False 검색 bool

> - **must**: bool must 절에 지정된 모든 쿼리가 일치하는 document 조회
> - **must_not**: bool must_not 절에 지정된 모든 쿼리가 일치하지 않는 document 조회
> - **should**: bool should 절에 지정된 모든 쿼리 중 하나라도 일치하는 document 조회
> - **filter**: filter절에 지정된 모든 쿼리와 일치하는 documnet를 조회 (score 무시)

```json
// bool.json
{
    "query": {
        "bool": {
            "must": [
                { "match": { "firstname": "Concetta" } }
            ],
            "must_not": [
                { "match": { "gender": "M" } }
            ]
        }
    }
}
```

- 중첩하기

```json
{
    "from": 0,
    "size": 1000,
    "query": {
        "bool": {
            "should": [ {
                    "bool": {
                        "must": {
                            "term": { "address": "street" }
                        },
                        "boost": 2.0
                    }
                }, {
                    "bool": {
                        "must": {
                            "term": { "address": "place" }
                        }
                    }
                }
            ]
        }
    }
  }
  
```
> boost : 가중치

### 조건절 검색 filter

```json
// filter.json
{
    "query": {
        "bool": {
            "filter": {
                "match": { "firstname": "Concetta" }
            }
        }
    }
}
```



### 범위 검색 range

> - gte: greater equal
> - gt: greater than
> - lte: less equal
> - ls: less

```json
// range.json
{
    "query": {
        "bool": {
            "filter": {
                "range": {
                    "balance": {
                        "gte": 10000,
                        "lte": 20000
                    }
                }
            }
        }
    }
}
```



### 역색인(Inverted Index) 검색 term, terms

역색인에 있는 토큰 중 키워드가 포함된 documnet를 조회

"a b c"라는 문자열은 ["a", "b", "c"] 로 역색인 되기 때문에 "a b c" 로는 검색이 안된다.

- term

```json
// term.json
{
    "query": {
        "term": {
            "address": "street"
        }
    }
}
```
* terms

  배열에 있는 키워드와 하나라도 일치하는 document 조회

```json
// terms.json
{
    "query": {
        "terms": {
            "address": ["street", "place"]
        }
    }
}
```



### 공통 옵션 size, from

```json
{
    "from": 0,
    "size": 3,
    "query": {
        "match_all": {}
    }
}
```

> **from**: sql의 offset
>
> **size**: sql의 limit

## 공통 옵션 sort

```json
{
    "sort": [
        { "age": "asc" },
        { "_score": "desc" }
    ],
    "query": {
        "match_all": {}
    }
}
```

> age에 대해 오름차순으로 정렬 후, _score에 대해 내림차순으로 정렬
>
> - 정렬을 사용하지 않을 경우 기본 정렬은 _score 내림차순

### 집계 하기 aggs (aggregations)

```json
{
    "query": {
        "term": {
            "address": "street"
        }
    },
    "aggs": {
        "balance_avg": { # response 에 명시할 통계 결과 필드명
            "avg" : { # 집계 타입 (sum, 등)
                "field" : "balance" # 어떤 필드를 통계 낼 것인지
            }
        }
    }
}
```

