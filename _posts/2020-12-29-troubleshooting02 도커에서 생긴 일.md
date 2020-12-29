---
layout: post
title: "[Trouble shooting] 2.도커에서 생긴 일"
description: "윈10의 WSL환경에서 도커를 사용하면서 생긴 일"
date: 2020-12-29 00:00:05
tags: [Trouble shooting, Docker, WSL]
comments: true
share: true
---

윈도우10에서 WSL을 이용해 도커를 사용하면서 생긴 오류

# 도커 오류

처음에 도커를 설치한 후에는 문제가 아무 문제가 없었으나, Docker가 업데이트 되고부터는 도커데몬이 실행중이 아니라는 오류 메시지가 출력되었다.

```bash
error during connect: Get http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.30/info: open //./pipe/docker_engine: The system cannot find the file specified. In the default daemon configuration on Windows, 
the docker client must be run elevated to connect. This error may also indicate that the docker daemon is not running.
```

아직 정확한 원인은 알지 못하였으나 ubuntu를 삭제 후 재 설치하여 해결하였다.



- ubuntu를 재설치 한 뒤 root 패스워드를 변경해주려면  파워쉘에서 실행한다.

```powershell
$ ubuntu1804.exe config --default-user root
# linux 접속 후
$ passwd
# 접속 해제 후 
$ ubuntu1804.exe config --default-user 유저
```

