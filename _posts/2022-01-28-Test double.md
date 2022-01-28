---
layout: post
title: "[Tdd] 테스트 더블"
description: "테스트 더블 종류 정리"
date: 2022-01-28 00:00:00
tags: [java, tdd, test double]
comments: true
share: true
---

# 테스트 더블
테스트를 진행하기 어려운 경우 테스트를 쉽게 진행하도록 만들어주는 객체

DB와 같은 인프라에 의존적인 서비스의 경우 테스트를 하기 위해 DB에 영향을 받기 때문에 상태에 따라 다른 결과가 나올 수 있기 때문에 이를 대신 사용할 수 있는 객체를 생성하여 대체한다.

## 테스트 더블의 종류

1. Dummy

    사용되지 않는 빈 객체. 어떻게 사용되는지 신경쓰지 않는 경우 사용된다. 생성자로 해당 객체를 받지만 테스트에서 사용되지 않기 때문에 null을 return 하거나 예외를 던지도록 한다.
    ```java
    public DummySomething implementation Something {
        @Override
        public void todo() {
            throw new RuntimeException();
        }
    }
    ```

2. Fake

    실제 행동을 모방하는 객체. 동작은 하지만 실제 동작과 다른 단순화된 로직을 구현한다.
    ```java
    public FakeSomething implementation Something {
        private List<Some> someList;

        ...

        @Override
        public Some getSomeById(long id) {
            return someList.get((int) id);
        }
    }
    ```


3. Stub

    테스트에서 호출된 요청에 대해 미리 준비된 결과를 반환해준다.
    

    ```java
    public StubSomething implementation Something {
        @Override
        public boolean isLogin() {
            return true;
        }
    }
    ```

4. Spy

    해당 객체가 사용되었는지 검증하기 위한 객체. Spy는 테스트에서는 Stub처럼 주입했지만, 테스트 종료 후에 검증을 하기 위해서도 사용된다.

    ```java
    public SpySomething implementation Something {
        public boolean isLogin_wasCalled;
        @Override
        public boolean isLogin() {
            isLogin_wasCalled = true;
            return true;
        }
    }
    ```

5. Mock (True mock)

    스스로를 검증할 수 있는 스파이 객체

    ```java
    public MockSomething implementation Something {
        public boolean login_wasCalled;
        @Override
        public void login() {
            login_wasCalled = true;
        }

        public void verify_loginned() {
            assertThat(login_wasCalled).isTrue();
        }
    }
    ```