---
layout: post
title: "[spring boot] Application Event"
description: "spring boot event 사용해보기"
date: 2021-03-25 00:00:00
tags: [spring boot]
comments: true
share: true
---

# Application Event

Spring에서는 Application Context에서 이벤트를 발생시키고 구독하는 기능을 제공한다.

Event를 사용하기 위해서는 3가지가 필수적이다.

1. 이벤트 발행자

   ​	이벤트를 발행하는 대상. 실제로 이벤트가 발행되는 것은 Application Context에서 이루어진다.

2. 이벤트 수신자

   ​	이벤트를 수신하는 대상. 어노테이션을 이용하는 방법과 ApplicationListener인터페이스를 구현하는 방법이 있다.

3. 이벤트

   ​	전달하기 위한 이벤트로 ApplicationEvent 상속받는다.

   