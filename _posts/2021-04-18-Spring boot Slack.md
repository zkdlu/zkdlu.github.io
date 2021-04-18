---
layout: post
title: "[spring boot] Slack으로 메시지 보내기"
description: "Slack을 이용해 다양한 메시지 보내보기"
date: 2021-04-18 00:00:00
tags: [spring boot, slack]
comments: true
share: true
---

# Slack

슬랙은 일반 메신저의 기능뿐만 아니라 슬랙에서 제공하는 Webhook을 이용한 메시지 전송이 가능함 

웹훅을 사용하기 위해 워크스페이스에 **Incoming WebHooks**를 추가해준다.

> https://api.slack.com/messaging/composing에 들어가면 슬랙에서 제공하는 기본적인 메시지 포맷이 있다.



### 기본방법 사용하기

Json 포맷

```json
{
    "username": "슬랙 봇 이름",
    "text": "전송 메시지 <link|표시 할 텍스트>",
    "icon_url": "아이콘"
}
```

Code

```java
void send() {
    Map<String, Object> payload = new HashMap<>();
    payload.put("username", "실험용 쥐");
    payload.put("text", "테스트용 메시지 <https://zkdlu.tistory.com|링크>");
    payload.put("icon_url", "https://avatars.githubusercontent.com/u/22608617?s=60&v=4");
    
    restTemplate.postForObject("web hook url", payload, String.class);
}
```



![기본](https://zkdlu.github.io/images/slack/default.png)









https://api.slack.com/block-kit

