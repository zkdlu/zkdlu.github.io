---
layout: post
title: "[spring boot] 멀티 모듈"
description: "spring boot프로젝트를 멀티 모듈로 만들어보자"
date: 2021-03-22 00:00:01
tags: [spring boot, intellij]
comments: true
share: true
---

# Spring boot 멀티 모듈 만들기

Intellij를 이용해 gradle 프로젝트를 생성 후 모듈을 추가한다.





```bash
Failed to apply plugin [id 'org.springframework.boot']
> Spring Boot plugin requires Gradle 5 (5.6.x only) or Gradle 6 (6.3 or later). The current version is Gradle 5.2.1
```

그동안 이런 오류가 떠서 멀티모듈을 구성하지 못하고 있었는데, 드디어 알았다.

로컬의 gradle을 사용하는게 아니라 gradle wrapper를 사용하기 떄문에 정상적으로 스크립트가 실행되지 못한것이다.

gradle wrapper의 버전을 업그레이드 시킨 후에 다시 해보자.

```bash
# intellij terminal
$ gradlew wrapper --gradle-version=6.8.3
```

> 빌드 스크립트가 정상적으로 실행되었다.