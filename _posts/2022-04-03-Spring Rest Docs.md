---
layout: post
title: "[spring boot] Spring Rest Docs"
description: "Spring Rest Docs 사용하기"
date: 2022-04-03 00:00:00
tags: [spring boot, document]
comments: true
share: true
---

### 1. 의존성 추가하기

```gradle

dependencies {
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}
```


### 2. asciidoc 파일을 build 폴더로 복사하기 위한 플러그인 추가

```gradle
plugins {
    id "org.asciidoctor.jvm.convert" version "3.3.2"
}
```

### 3. gradle task 지정

```gradle
ext{
    snippetDir = file('build/generated-snippets')
}

test {
    useJUnitPlatform()
    outputs.dir snippetDir
}

asciidoctor {
    inputs.dir snippetDir
    dependsOn test
}

task copyDocument(type: Copy) {
    dependsOn asciidoctor

    from file("build/docs/asciidoc/")
    into file("src/main/resources/static/docs")
}

bootJar {
    dependsOn copyDocument

    from file('src/main/resources/static/docs')
    into file('build/resources/main/static/docs')
}

```

### 4. document 작성

src/docs/asciidoc 하위에 adoc 파일 작성

```adoc
= Spring REST Docs
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

ifndef::snippets[]
:snippets: ./build/generated-snippets
endif::[]


[[Member_API]]
== Demo API

[[Demo_조회]]
=== Demo 조회

include::{snippets}/demo-api-test/get-demo/http-request.adoc[]
include::{snippets}/demo-api-test/get-demo/http-response.adoc[]
```