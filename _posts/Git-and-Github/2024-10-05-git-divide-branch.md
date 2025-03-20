---
title: "[Git] #4-2. 브랜치 나누기"
excerpt: "브랜치 나누기 이론 이해하기"

categories:
  - Git
tags:
  - [Git, Branch]

permalink: /theory/git/divide-branch

toc: true
toc_sticky: true

date: 2024-10-05
last_modified_at: 2024-10-05
---

## 브랜치

- 브랜치 이름: 최초의 브랜치, master 브랜치(github에서는 main 브랜치라고 한다)
- 가장 기본적인, 최초의 브랜치이다. 지금까지 만든 커밋들은 모두 기본적으로 master 브랜치에 속해 있다.


예를 들어 이 그림을 보자. 

![image](https://github.com/user-attachments/assets/5ac5c824-d4d4-4e88-b3a7-f327529c7f3d)

master 브랜치에는 커밋이 <span style="color:red"> 세 개 </span>, foo 브랜치에는 커밋이 <span style="color:red"> 다섯 개 </span> 쌓여 있다.

![image](https://github.com/user-attachments/assets/18352b97-9dfb-47b8-9314-8af35de2a616)

master 브랜치에는 커밋이 <span style="color:red"> 세 개 </span>, foo 브랜치에는 커밋이 <span style="color:red"> 다섯 개 </span>, bar 브랜치에는 커밋이 <span style="color:red"> 여섯 개 </span> 쌓여 있다.

---

### 브랜치 이름을 어떻게 지을까? 

![image](https://github.com/user-attachments/assets/d72cf3a0-cbbb-4624-9140-b4814aadba46)

강제하진 않지만 막 짓는 것보다 목적에 맞게 브랜치 이름을 설정하는 게 좋다.

---

### 특정 브랜치에서 작업하기 : HEAD와 체크아웃

- <span style="background-color:#fff5b1"> HEAD </span>
  - 현재 작업 중인 브랜치의 커밋을 가리킨다.
  - 일반적으로 현재 작업 중인 브랜치의 <span style="color:red"> 최신 커밋 </span>을 가리킨다.
  - 한 마디로 "내가 지금 어디에서 작업 중인가"를 가리킨다.

![image](https://github.com/user-attachments/assets/51ae9af8-b3bf-4671-af69-c55b8b6b99c4)

(연결리스트가 떠오른다...)

HEAD가 bar 브랜치의 6번 커밋을 가리키고 있다 ➡️ 현재 여섯 개의 커밋이 쌓여 있고, 최신 커밋은 bar 6번 커밋이다.

HEAD를 요리조리 옮겨서 작업 브랜치를 바꿀 수 있는데, 이를 체크 아웃이라고 한다.


- <span style="background-color:#fff5b1"> 체크 아웃(checkout) </span>
  - 특정 브랜치에서 작업할 수 있도록 환경을 바꾸는 것이다.
  - HEAD의 위치를 특정 브랜치의 최신 커밋으로 옮긴다.

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*