---
layout: post
title: "[spring boot] Spring Rest Docs 더 잘 쓰기"
description: "Spring Rest Docs 사용하기"
date: 2022-04-20 00:00:00
tags: [spring boot, document]
comments: true
share: true
---

### Root 문서에서 하위 문서 참조하기

spring rest docs는 xref를 사용해 다른 adoc 파일 링크를 걸 수 있다.

```adoc
// index.adoc
= API Document
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectlinks:

- xref:member.adoc[Member APIs]
```

아래와 같이 operation을 사용하면 snippets에 있는 문서를 추가한다.

```adoc
// member.adoc
= Member APIs
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:

== Member API

operation::member-integration-test/get-my-self[snippets='curl-request,http-request,http-response']
```


## 안되네요?

intellij에서는 operation 이 잘 동작하지만, 빌드 후 서버를 실행해보면 정상동작 하지 않는 문제가 있었다.

org.asciidoctor.jvm.convert 플러그인에서는 동작하지 않는 문제가 있는 것 같다.

build.gradle에 아래의 내용을 추가해준다.


```groovy
// build.gradle
configurations {
    asciidoctorExt
}

dependencies {
    ...
    asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
}

asciidoctor {
    configurations 'asciidoctorExt'
    ...
}
```


[참고] [https://github.com/spring-projects/spring-restdocs/issues/680](https://github.com/spring-projects/spring-restdocs/issues/680)