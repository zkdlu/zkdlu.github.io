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

처음 gradle 프로젝트를 생성하면 다음과 같은 디렉토리 구조를 가진다.

```
📦multimodule
 ┣ 📂gradle
 ┣ 📂src
 ┣ 📜build.gradle
 ┣ 📜gradlew
 ┣ 📜gradlew.bat
 ┗ 📜settings.gradle
```

## 1. 새 모듈 추가

root 프로젝트에서 새로운 Gradle 모듈을 추가합니다.

이번 프로젝트는 간단하게 module-common과 module-api로 하여  각각 추가합니다.

멀티 모듈 프로젝트는 root 프로젝트에서 빌드가 이루어지기 때문에 새로 추가하는 모듈에는 src 파일과 build.gradle만 가지게 됩니다.

우선 추가 된 모듈의 build.gradle을 전부 지워줍니다.

## 2. settings.gradle

```groovy
rootProject.name = 'multimodule'
include 'module-common'
include 'module-api'
```

새로운 하위 프로젝트를 추가하면 root 에 있는 settings.gradle에 다음과 같이 추가 되는걸 볼 수 있습니다.

이는 rootProject인 'multimodule'이 'module-common', 'module-api' 프로젝트를 관리하겠다는 의미입니다.

## 3. build.gradle

```groovy
buildscript {
    ext {
        springBootVersion = '2.4.4'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    group 'com.zkdlu'
    version '1.0-SNAPSHOT'

    sourceCompatibility = 11

    repositories {
        mavenCentral()
    }
    dependencies {
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
}

project(':module-common') {
    dependencies {
    }
}
project(':module-api') {
    dependencies {
        compile project(':module-common')
    }
}
```

- subproject

settings.gradle에 명시된 하위 프로젝트들에 사용될 플러그인과 의존성 등 빌드 설정을 합니다.

- project(:  - )

해당 프로젝트의 의존성을 추가합니다.

### 💡Trouble shooting

```bash
Failed to apply plugin [id 'org.springframework.boot']
> Spring Boot plugin requires Gradle 5 (5.6.x only) or Gradle 6 (6.3 or later). The current version is Gradle 5.2.1
```

build.gradle을 멀티모듈에 맞게 설정하면 이런 오류가 뜨는데, 로컬의 gradle을 사용하는게 아니라 gradle wrapper를 사용하기 떄문에 설치된 gradle 버전이 적용되지 않은 것이다.

gradle wrapper의 버전을 업그레이드 시킨 후에 다시 해보자.

```bash
# intellij terminal
$ gradlew wrapper --gradle-version=6.8.3
```

> 빌드 스크립트가 정상적으로 실행되었다.



## 4. 모듈 작성

> 기본 패키지를 생성 후 소스코드를 작성해야 한다.



각 모듈별로 필요한 의존성을 추가 한 후 코드를 작성한다.



### module-common

```groovy
# module-common/build.gradle

dependencies {

}
```



### module-api

```groovy
# module-api/build.gradle

dependencies {

}
```

