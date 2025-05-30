---
title: "[Git] #1-2. 깃이 없는 세상: 버전과 버전 관리 "
excerpt: "깃이 필요한 이유와 버전 이해하기"

categories:
  - Git
tags:
  - [Git, Version]

permalink: /theory/git/version

toc: true
toc_sticky: true

date: 2024-09-24
last_modified_at: 2024-09-24
---

## 깃을 왜 배워야 할까?

깃이 없으면? 
1. 변경 내역 확인이 어렵다. - 파일이 어떤 과정을 거쳐 지금 모습이 됐는지, 최종본이 되기까지 어떤 변경 사항을 거쳤는지 확인하기 어렵다.
2. 작업을 되돌리기 어렵다. - 파일의 여러 변경 사항(삭제, 수정, 추가 등)을 되돌릴 때
3. 협력하기 어렵다. - 각자의 개발 내용을 일일이 다 확인하고 비교해야 해서 시간도 많이 걸리고, 실수도 빈번하게 일어난다.

→ 이러한 어려움을 느낀 Linus Torvalds, Linux 운영체제는 2700만 줄이 넘는 분량의 코드이고 오픈 소스라 코드가 시시각각 변화하는데 프로그램의 변경 사항을 확인하고, 작업 되돌리고, 협업하기 어려워 Git을 만들었다.

---

## 버전이란?

- 버전이란, 유의미한 변화가 결과물로 나온 것이다.
- 프로그램 개발은 유의미한 변화를 쌓아 프로그램을 만들어나가는 것이다.
- 예시. 회원 가입 기능, 버그 수정, 메뉴 바 완성, 결제 기능 등
- 버전 관리란 변경 내역들을 기억하며, 필요하다면 작업을 되돌리고 여러 명의 코드를 쉽게 나누고 합치며 개발하는 것이다.
- Git은 버전 관리를 위한 도구이다.

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*