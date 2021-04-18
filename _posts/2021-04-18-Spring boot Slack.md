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

> [slack message](https://api.slack.com/messaging/composing)에 들어가면 슬랙에서 제공하는 기본적인 메시지 포맷이 있다.



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





### 반응형 메시지 사용하기

슬랙에서 제공하는 [Block Kit](https://api.slack.com/block-kit) 을 이용해서 슬랙으로 투표같은 기능을 만들 수 있다.



1. [https://api.slack.com/](https://api.slack.com/) 에 접속하여 App을 생성해준다.

   앱을 사용해주기 위해 App Home탭에서 봇 유저를 설정해준다.

   ![bot user](https://zkdlu.github.io/images/slack/botuser.png)

   

2. API 사용하기 위한 WebHook과 상호작용을 위한 URL을 설정해준다.

   각각 Incoming Webhooks와 Interactivity & Shortcuts에서 설정이 가능하다.

   Interactivity의 Request URL은 슬랙 메시지에서 특정 Action을 했을 경우 



3. Action을 정의해준다.

   특정 Action을 하기 위한 Shortcuts를 생성해준다.

   

   Shortcut에 등록하는 Action 정보

   - Name : Action 이름
   - Short Description : Action 설명
   - Callback ID : Action ID

   ![short cut](https://zkdlu.github.io/images/slack/shortcut.png)



4. Slack에서 제공하는 SDK를 추가한다. [https://github.com/slackapi/java-slack-sdk ](https://github.com/slackapi/java-slack-sdk)

   ```groovy
   dependencies {
       implementation 'com.squareup.okhttp3:okhttp:4.2.0'
       implementation 'com.slack.api:slack-api-model:1.0.0-RC2'
       implementation 'com.slack.api:slack-api-client:1.0.0-RC2'    
   }
   ```

   

...작성중....

