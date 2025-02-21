---
title: "[Git] #4-4. 브랜치 합치기"
excerpt: "브랜치 합치기 이론 이해하기"

categories:
  - Git
tags:
  - [Git, CombineBranch, Branch]

permalink: /theory/git/combine-branch

toc: true
toc_sticky: true

date: 2024-10-07
last_modified_at: 2024-10-07
---

## 브랜치 합치기

브랜치를 합친다 == 브랜치를 <span style="color:red"> 병합(merge) </span>한다.

---

### 빨리 감기 병합(fast-forward merge) 

foo 브랜치를 master 브랜치로 병합하면?

![image](https://github.com/user-attachments/assets/32e60b1f-8f95-4bb3-9e30-fb195915d3be) | ![image](https://github.com/user-attachments/assets/7e08acbf-865a-4c67-8c5c-0e67a7b2584d)

foo 브랜치가 뻗어 나오고, foo 브랜치에 커밋이 쌓이고, 병합되는 순간까지 <span style="color:red"> master 브랜치는 가만히 있었다. </span>

master 브랜치는 <span style="color:red"> 마치 빨리감기하듯 </span> 그저 foo 브랜치에서 추가된 커밋을 반영하기만 하면 된다.

➡️변함이 없던 브랜치가 <span style="color:red"> 마치 빨리 감기 하듯 </span> 브랜치 내용이 업데이트되는 병합 기법을 <span style="color:red"> 빨리 감기 병합(fast-forward merge) </span>라고 한다.

---

### 빨리 감기 병합이 아닌 병합은?

bar 브랜치에는 없는 커밋이 master 브랜치에 있고, master 브랜치에 없는 커밋이 bar 브랜치에 있는 상황 

![image](https://github.com/user-attachments/assets/cbc5ffbb-3cd0-4f72-bb24-0bf0a33f5554)


두 브랜치를 병합한 <span style="color:red"> 새로운 커밋 </span>이 생성된다.

![image](https://github.com/user-attachments/assets/4234443d-ff7c-41e6-880a-503da1978b04)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*