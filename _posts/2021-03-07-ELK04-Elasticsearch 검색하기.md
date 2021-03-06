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



## URI Search

URL에 파라미터를 넘기는 방법

```bash
http://localhost:9200/account/_search?q=gender:F&size=1&from=1
```

> 저장된 계좌정보중에서 성별이 여자인 결과를 1번째부터 1개만 가져오기 (기본은 0번째부터)



## Query DSL

json 파일에 쿼리를 작성하여 POST로 넘기는 방법. 더욱 상세한 표현이 가능



## Response

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
                "_type": "_doc", // 7.x부터 _doc으로 통일,
                "_id": "id",
                "_score": "해당 document가 검색 쿼리와 일치하는 상대적인 값",
                "_source": {
                    // 데이터
                }
            }
        ]
    }
}
```

