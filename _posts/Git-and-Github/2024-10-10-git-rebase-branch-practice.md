---
title: "[Git] #4-7. 브랜치 재배치하기"
excerpt: "브랜치 재배치 실습"

categories:
  - Git
tags:
  - [Git, Branch, Rebase]

permalink: /theory/git/branch-rebase-pracitce

toc: true
toc_sticky: true

date: 2024-10-10
last_modified_at: 2024-10-10
---

## 브랜치 재배치란?

master 브랜치의 두 번째 커밋에서 foo 브랜치가 뻗어 나왔고, 각 브랜치에 여러 커밋이 쌓여 있다.

![image](https://github.com/user-attachments/assets/cf6a3649-6921-43a3-adf4-342620e090d3)

두 번째 커밋에서 뻗어나온 foo 브랜치를 네 번째 커밋에서 뻗어나오도록 변경한다.

![image](https://github.com/user-attachments/assets/ee0c076c-9cc0-4ca0-92f0-bb14ee761161)

&nbsp;

이렇게 브랜치가 뻗어나온 기준점을 변경하는 것을 <span style="color:red"> 브랜치의 재배치(rebase) </span> 라고 한다.

---

### 브랜치 재배치 실습

앞의 예시 그래프대로 실습을 해보자.

![image](https://github.com/user-attachments/assets/cf6a3649-6921-43a3-adf4-342620e090d3)

빈 저장소에 a.txt를 생성한 커밋(first master commit), b.txt를 생성한 커밋(second master commit)을 한 후 foo 브랜치를 만들고 체크아웃 한다. 

foo 브랜치에 foo_a.txt를 생성 후 커밋(first foo commit)하고, foo_b.txt를 생성 후 커밋(second foo commit)한다.

이제 master 브랜치로 체크아웃한다. master 브랜치에서 c.txt를 생성 후 커밋(third master commit)하고, d.txt를 생성 후 커밋(fourth master commit)한다.

&nbsp;

그러면 이런 그래프가 나타난다.

![image](https://github.com/user-attachments/assets/3f5e33eb-4c97-4e99-a112-be1ff3466511)


&nbsp;

이제 두 번째 커밋에서 뻗어 나온 foo 브랜치를 네 번째 커밋에서 뻗어나오도록 재배치(rebase) 해보자.

![image](https://github.com/user-attachments/assets/ee0c076c-9cc0-4ca0-92f0-bb14ee761161)

foo 브랜치로 체크아웃한 후, 재배치하고자 하는 커밋(fourth master commit)에 마우스 우클릭 - 재배치를 누른다.

&nbsp;

그러면 이렇게 브랜치가 재배치(rebase)되었다!

![image](https://github.com/user-attachments/assets/2fdc8ef5-2832-406e-a4d0-be1297b7c055)

---

### 재배치했을 때 발생하는 충돌

재배치를 했을 때 같은 부분을 다르게 수정해서 충돌이 발생할 수 있다. 이럴 땐 앞서 충돌을 해결한 방법과 똑같이 하면 된다.

충돌이 발생했을 때 대처법 
1. 충돌을 해결한다(어떤 브랜치의 내용을 반영할지 직접 선별한다).
2. 다시 커밋한다.

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*