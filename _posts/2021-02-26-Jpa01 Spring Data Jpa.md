---
layout: post
title: "[Jpa] Spring Data Jpa"
description: "Spring Data Jpa 사용하기"
date: 2021-02-26 00:00:00
tags: [spring boot, jpa]
comments: true
share: true
---







데이터 소스 설정

```properties
# 기본 설정 (h2, hsql의 메모리 db는 아무 설정 안해도 됨)
spring.datasource.url=jdbc:[db]://[host]:[db-name]
spring.datasource.username=zkdlu
spring.datasource.password=1234

# 생성되는 sql 확인용
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# ? 로 출력되는거 로깅으로 확인하기
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql=trace
```



