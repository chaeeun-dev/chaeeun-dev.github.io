---
title: "[Git] #4-1. 브랜치란: 브랜치를 나누어 관리하는 이유"
excerpt: "브랜치 이해하기"

categories:
  - Git
tags:
  - [Git, Branch]

permalink: /theory/git/branch-theory

toc: true
toc_sticky: true

date: 2024-10-04
last_modified_at: 2024-10-04
---

## 브랜치란?


브랜치 : 버전을 여러 개의 흐름으로 관리하는 방법, <span style="background-color:#fff5b1"> 버전의 분기 </span>

![image](https://github.com/user-attachments/assets/9a848bfd-a02f-4ea7-8b6d-7bbab4e0c58d)

---

### 브랜치가 없다면? 

**[예시1]**

![image](https://github.com/user-attachments/assets/f6b9b997-d40b-4026-943b-4d685f014c41) | ![image](https://github.com/user-attachments/assets/9cd0e7b5-b54b-4c82-ab00-38ca9ee7d00d)


- 서로의 작업과 전혀 관련 없는 부분, 같은 코드를 다르게 수정한 부분 혼재
- 일일이 수작업으로 합쳐야함
- 때로는 서로의 코드를 합치다 실수가 생길수도

---

**[예시2]**

C 회사가 제작한 프로그램을 여러 회사에 납품한다.

![image](https://github.com/user-attachments/assets/022714a6-569d-4f5d-956d-cfedb051652a)

프로그램을 사용하는 회사들은 C 회사에 각자의 요구사항을 전달한다. C 회사는 각 회사의 요구 사항의 수만큼 코드를 복사하고, 수정해야 한다. 

![image](https://github.com/user-attachments/assets/bdea1642-99cf-416e-9c84-dea9257498b8)

시간이 흘러 프로그램은 v10.0.0까지 출시했고, 요구 사항은 1000개가 넘어갈 정도로 많아졌다. (v1.0.0에 대한 요구 사항 30개, v.3.0.0에 대한 요구 사항 80개 등) 버전별로 복사본을 만들고, 요구 사항별로 복사본을 만들어 요구 사항에 맞게 수정해야 할까? 매우 번거로운 일이다.

![image](https://github.com/user-attachments/assets/ff5175b4-b13b-43d1-8900-22195fb3e264)

---

### 브랜치로 문제 해결하기

브랜치는 버전의 분기이다.

- 브랜치로 버전의 분기를 관리하는 방법

![image](https://github.com/user-attachments/assets/5fd30c62-5cf9-4cdf-8e33-b8cfd4e8d57c)

1. 브랜치를 <span style="color:red"> 나눈다 </span>
2. 각자의 브랜치에서 <span style="color:red"> 작업한다 </span>
3. (필요하다면) 나눈 브랜치를 <span style="color:red"> 합친다 </span>

---

**[예시1 문제 해결]** 

- 브랜치를 나눈다.

![image](https://github.com/user-attachments/assets/6b0e52aa-4084-4013-8698-f6ca4aba0724)

- 각자의 브랜치에서 작업한다.

![image](https://github.com/user-attachments/assets/72bec97e-ac88-4cf2-94e5-dba9b5c8d52d)

- 나눈 브랜치를 합친다.

![image](https://github.com/user-attachments/assets/a823d858-ff06-4cf6-9042-ccf0fa72577f)

내가 작업한 영역에서 다른 사람이 수정한 게 있는지만 확인하면 된다. 다른 사람이 작업한 코드 중에서 내 영역이 아니면 굳이 볼 필요가 없다.

→ 이제 <span style="color:red"> 같은 부분을 다르게 수정한 부분 </span>만 보면 된다(깃이 알려준다!).

---

**[예시2 문제 해결]** 

![image](https://github.com/user-attachments/assets/a045b4dd-69ee-497b-b40a-9363f57b9352)


버전 별로 출시하고, 각 버전에 대한 요구 사항이 생기면 그때그때 브랜치를 나눠 수정하면 된다. 복사본을 일일이 만드는 것 보다 메모리도 적게 들고, 실수도 줄일 수 있고, 작업도 훨씬 간편해진다.

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*