---
title: "[Server] #15-9. 이벤트와 조건 변수"
excerpt: "Producer & Consumer 패턴으로 이벤트와 조건 변수 활용하기"

categories:
  - Server
tags:
  - [Server, Event, ConditionVariable]

permalink: /theory/server/event-and-condition-variable/

toc: true
toc_sticky: true

date: 2024-11-13
last_modified_at: 2025-03-14
---

## 이벤트와 조건 변수

멀티 스레드 환경에서 데이터를 안전하게 공유하면서 대기 시간을 줄이는 방법을 알아본다. 이번에는 Producer & Consumer 패턴을 사용하여 이벤트(Event)와 조건 변수(Condition Variable)을 활용하는 방법을 알아보자.

---

### Producer & Consumer 패턴

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*