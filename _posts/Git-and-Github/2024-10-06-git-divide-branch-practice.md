---
title: "[Git] #4-3. 브랜치 나누기"
excerpt: "브랜치 나누기 실습"

categories:
  - Git
tags:
  - [Git, Branch]

permalink: /theory/git/divide-branch-practice

toc: true
toc_sticky: true

date: 2024-10-06
last_modified_at: 2024-10-06
---

## 브랜치 나누기 실습

master 브랜치에 커밋 두 개를 만든다.

(강의에서는 master라고 나와 있는데 나는 main 브랜치로 뜬다. 상관은 없는 것 같다.)

![image](https://github.com/user-attachments/assets/2f2b3fc5-dc54-4e5b-9c2c-61a40c6d7bc6)

&nbsp;

상단에 있는 브랜치를 누르고, 새 브랜치 이름을 입력한다. 

![image](https://github.com/user-attachments/assets/986eff07-077f-4d45-99b7-faa4fda354e7)

📌여기서 '새 브랜치 체크아웃'에 체크를 하면, foo 브랜치를 만듦과 동시에 작업 환경을 foo로 바꾸겠다는 것이다.

&nbsp;

![foo_branch](https://github.com/user-attachments/assets/75a5145c-2a04-4566-98c4-4c2807603f17)

이렇게 foo 브랜치가 생성됨과 동시에 체크아웃 되었는데, 볼드체와 동그라미가 보인다.

&nbsp;

![image](https://github.com/user-attachments/assets/453e69e1-7f50-41a5-a6b1-3ab70d0c3304)

(foo 브랜치에서)작업 디렉터리에 foo_a.txt를 생성하고, commit한다.

&nbsp;

![image](https://github.com/user-attachments/assets/3fe196f5-9a9d-4adf-b24e-d75e84def732)

foo 브랜치에는 커밋이 3개 쌓여 있고(파일도 3개), master 브랜치에는 커밋이 2개 쌓여 있다(파일도 2개, foo_a.txt 파일이 없음).

&nbsp;

그럼 master 브랜치로 체크아웃 해보자. 다른 브랜치로 체크아웃하려면 그 브랜치에 더블 클릭하면 된다. 그러면 HEAD도 이동하게 된다.

![image](https://github.com/user-attachments/assets/d34e0fa5-06d4-4a1b-93c9-c4df0b000518)

&nbsp;

탐색기를 열어보면, 파일이 두 개 있다는 걸 알 수 있다.

![image](https://github.com/user-attachments/assets/df3d3c10-0d1b-4c95-a163-e5cbdf0d2b17)

브랜치 나누기 실습 끝!

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*