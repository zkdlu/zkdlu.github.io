---
layout: post
title: "[Trouble shooting] 11.ECS 태스크 실행이 안돼요."
description: "AWS ECS 사용하기"
date: 2022-07-10 00:00:00
tags: [spring boot, aws, ecs]
comments: true
share: true
---

간단한 Spring Boot App을 도커 이미지로 빌드 후 ECS에 배포하려 했는데 태스크가 Pending되며 종료되었다.


## 원인

```bash
WARNING: The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64) and no specific platform was requested
standard_init_linux.go:228: exec user process caused: exec format error
```

M1 맥에서 빌드할 때 생성된 빌드 플랫폼과 EC2 서버와 호환성이 안맞는 문제였다 ㅎㅎ;;


## 해결

```bash
docker build --platform linux/amd64 -t zkdlu/test:v4 .
```

빌드시 플랫폼 지정하여 빌드한다