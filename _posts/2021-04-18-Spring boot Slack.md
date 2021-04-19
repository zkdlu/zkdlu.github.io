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



1. **[https://api.slack.com/](https://api.slack.com/) 에 접속하여 App을 생성해준다.**

   앱을 사용해주기 위해 App Home탭에서 봇 유저를 설정해준다.

   ![bot user](https://zkdlu.github.io/images/slack/botuser.png)

   

2. **API 사용하기 위한 WebHook과 상호작용을 위한 URL을 설정해준다.**

   각각 Incoming Webhooks와 Interactivity & Shortcuts에서 설정이 가능하다.

   Interactivity의 Request URL은 슬랙 메시지에서 특정 Action을 했을 경우 



3. **Action을 정의해준다.**

   특정 Action을 하기 위한 Shortcuts를 생성해준다.

   

   Shortcut에 등록하는 Action 정보

   - Name : Action 이름
   - Short Description : Action 설명
   - Callback ID : Action ID

   ![short cut](https://zkdlu.github.io/images/slack/shortcut.png)



4. **Slack에서 제공하는 SDK를 추가한다. [https://github.com/slackapi/java-slack-sdk ](https://github.com/slackapi/java-slack-sdk)**

   ```groovy
   dependencies {
       implementation 'com.google.code.gson:gson:2.8.6'
       implementation 'com.squareup.okhttp3:okhttp:4.2.0'
       implementation 'com.slack.api:slack-app-backend:1.6.2'
       implementation 'com.slack.api:slack-api-model:1.6.2'
       implementation 'com.slack.api:slack-api-client:1.6.2'
   }
   ```

5. **Slack 메시지를 조합할 수 있는 메서드를 만든다.**

   Slack 의 메시지 레이아웃은 LayoutBlock을 상속받아 구현되어 있다.

   > [LayoutBlock](https://api.slack.com/reference/block-kit/blocks)에서 다양한 블록을 확인할 수 있음.

   ```java
   private LayoutBlock getHeader(String text) {
       return Blocks.header(h -> h.text(
           BlockCompositions.plainText(pt -> pt.emoji(true)
                                       .text(text))));
   }
   
   private LayoutBlock getSection(String message) {
       return Blocks.section(s -> s.text(
           BlockCompositions.markdownText(message)));
   }
   
   private BlockElement getActionButton(String plainText, String value, String style, String actionId) {
       return BlockElements.button(b -> b.text(plainText(plainText, true))
                                   .value(value)
                                   .style(style)
                                   .actionId(actionId));
   }
   ```

6. **Slack으로 보내 줄 투표 메시지를 조합하여 보내준다.**

   ```java
   private List<BlockElement> getActionBlocks() {
       List<BlockElement> actions = new ArrayList<>();
       actions.add(getActionButton("확인", "ok", "primary", "action_success"));
       actions.add(getActionButton("취소", "fail", "danger", "action_fail"));
       return actions;
   }
   
   public String vote(String message) throws IOException {
       List<LayoutBlock> layoutBlocks = Blocks.asBlocks(
           getHeader("골라 주세요!"),
           Blocks.divider(),
           getSection(message),
           Blocks.divider(),
           Blocks.actions(getActionBlocks())
       );
   
       Slack.getInstance().send("slack app web hook", WebhookPayloads
                                .payload(p -> p.text("골라 골라~")
                                         .blocks(layoutBlocks)));
   
       return message;
   }
   ```

   슬랙에서 전송된 메시지를 확인한다.

   ![vote](https://zkdlu.github.io/images/slack/vote.png)

7. **슬랙에서 버튼을 누르면 설정 페이지에서 지정해둔 Request URL로 Callback이 온다. **

   슬랙에서 보내오는 payload는 Content Type이 Application Form UrlEncoded으로  Post 메서드를 사용한다.

   @RequestParam 어노테이션을 사용해 payload를 수신하고 Gson 라이브러리를 사용해 BlockActionPayload 타입으로 변환해준다.

   ```java
   @PostMapping(
       value = "/slack/callback", 
       consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
   public String callback(@RequestParam String payload) throws IOException {
       var blockPayload = GsonFactory.createSnakeCase()
                   .fromJson(payload, BlockActionPayload.class);
       
       return slackBot.callbackVote(blockPayload);
   }
   ```

   BlockActionPayload를 보면 유저, 선택한 Button 등의 정보가 들어 있다.

8. **슬랙이 보내온 Payload에서 유저와 선택한 버튼정보를 저장하고 전체 정보를 집계 해서 보내준다.**

   예제에서는 DB 연동없이 임의로 static 객체를 사용한다.

   

   반환하는 메시지의 레이아웃은 슬랙 예제를 따라하였다.

   [slack 예제](https://app.slack.com/block-kit-builder/T01TNPY6W1E#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22New%20request%22,%22emoji%22:true%7D%7D,%7B%22type%22:%22section%22,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Type:*%5CnPaid%20Time%20Off%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Created%20by:*%5Cn%3Cexample.com%7CFred%20Enriquez%3E%22%7D%5D%7D,%7B%22type%22:%22section%22,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*When:*%5CnAug%2010%20-%20Aug%2013%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Type:*%5CnPaid%20time%20off%22%7D%5D%7D,%7B%22type%22:%22section%22,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Hours:*%5Cn16.0%20(2%20days)%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Remaining%20balance:*%5Cn32.0%20hours%20(4%20days)%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22%3Chttps://example.com%7CView%20request%3E%22%7D%7D%5D%7D)

   ```java
   private static Set<Vote> votes = new HashSet<>();
   
   private TextObject getField(Vote vote) {
       return BlockCompositions.markdownText(
           "*" + vote.getUser() + "*\n" +
           (vote.getActionId().equals("action_success") ? "동의" : "거부"));
   }
   
   private LayoutBlock getFieldSection(List<TextObject> fields) {
       return Blocks.section(s -> s.fields(fields));
   }
   
   public String callbackVote(BlockActionPayload blockPayload) throws IOException {
       var user = blockPayload.getUser().getUsername();
       var actionId = blockPayload.getActions().get(0).getActionId();
   
       Vote vote = new Vote(user, actionId);
       votes.add(vote);
   
       var fields = votes.stream()
           .map(this::getField)
           .collect(Collectors.toList());
   
       List<LayoutBlock> blocks = Blocks.asBlocks(
           getHeader("집계 결과"),
           Blocks.divider(),
           getFieldSection(fields)
       );
   
       ActionResponse response = ActionResponse.builder()
           .replaceOriginal(false)
           .blocks(blocks)
           .build();
   
       ActionResponseSender sender = new ActionResponseSender(Slack.getInstance());
       sender.send(blockPayload.getResponseUrl(), response);
   
       return user;
   }
   ```

   > ActionResponse의 replaceOriginal 을 true로 할 경우 기존의 메시지를 새로 보내는 메시지로 교체한다.

   ![vote-result](https://zkdlu.github.io/images/slack/vote-result.png)
