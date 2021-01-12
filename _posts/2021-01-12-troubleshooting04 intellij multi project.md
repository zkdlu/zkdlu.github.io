---
layout: post
title: "[Trouble shooting] 4.intellij multi module 사용하기"
description: "intellij에서 멀티모듈을 사용하면서 발생한 오류"
date: 2021-01-12 00:00:03
tags: [Trouble shooting, intellij]
comments: true
share: true
---

### 무엇을 했는가?

intellij 에서 여러개의 spring boot app을 실행시키기 위해 다음 과 같이 진행하였다.

1. intellij에서 빈 프로젝트 생성
2. module을 새로 추가하여 spring boot app을 생성 후 실행



### 어떤일이 있었는가?

처음에는 **Configuration is still incorrect** 라는 오류가 뜨고 패키지를 찾을 수 없다고 하였다.

인텔리제이 버그인거 같다. 인텔리제이 특유의 인덱싱 하는 버그가 2020.02에서는 업데이트 되었다고 하는데 나는 2019인걸... 

프로젝트를 re-load하니 이 문제는 해결 되었길레 실행을 해보았는데 이번엔 

**Error: Java: Invalid source release: 11**

라는 오류가 발생하였다. 

프로젝트는 jdk 11로 빌드되었는데 실행을 다른 버전의 jdk를 사용하면 나오는 문제이다.



### 어떻게 하였는가?

먼저 인텔리제이에 설정 된 SDK버전을 확인해보았다.

- > File > Settings > Build, Execution, Deployment > Compiler > Java Compiler

  Project bytecode version이 11이다.  통과..

멀티 모듈을 사용했으니까 모듈 설정이 문제인가?

- > File > Project Structure  > Project Settings > Modules > Module 선택

  Language level이 11이다.  통과..

멀티 모듈을 만들기 위해 만든 빈 프로젝트가 문제인가??

- > File > Project Structure > Project Settings > Project > Project SDK & Language level

  이놈이 문제였다. 빈 프로젝트를 생성하면서 jdk 버전을 선택하지 않았는데 기본 설정이 1.8이었다.

  11로 변경해주니 정상 동작한다.



프로젝트 하위에 있는 각 모듈들이 독립적으로 실행이 되는데

인텔리제이는 동작을 어떻게 하길레 프로젝트 버전과 모듈의 버전이 맞아야 하는가..?




