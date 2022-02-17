---
layout: post
title: "[Gradle] 로컬 Maven Repository에 저장하기"
description: "jar파일 로컬 maven reposiory에 저장하기"
date: 2022-02-18 00:00:00
tags: [java, gradle]
comments: true
share: true
---

## 0. Mac maven repository 경로

```bash
$ ~/.m2/repository
```

## 1. build.gradle에 maven plugin 및 group, version 추가

```groovy
plugins {
    ...
    id 'maven'
}

group 'com.zkdlu'
version '1.0-SNAPSHOT'

```

> gradle 버전 7부터는 "maven-publish"로 플러그인 명칭이 변경되었다.


## 2. build.gradle에 Component 설정

```groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
````

java(jar)와 web(war) component를 사용할 수 있다.

## 3. Local Maven Repository에 배포하기

```groovy
$ ./gradlew publishToMavenLocal
```

intellij 의 gradle 탭에서도 가능

## 4. 사용하기

```groovy
repositories {
    mavenLocal()
}

implementation 'com.zkdlu:my-library:1.0-SNAPSHOT''
```