---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션4 브랜치로 나누어 관리하기"
date: 2024-10-05 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

## [이론] 브랜치란 : 브랜치를 나누어 관리하는 이유(24.10.05)

![image](https://github.com/user-attachments/assets/9a848bfd-a02f-4ea7-8b6d-7bbab4e0c58d)


브랜치 : 버전을 여러 개의 흐름으로 관리하는 방법, <span style="background-color:#fff5b1"> 버전의 분기 </span>


### 브랜치를 사용하는 이유 : 브랜치가 없다면? 

**[예시1]**

![image](https://github.com/user-attachments/assets/f6b9b997-d40b-4026-943b-4d685f014c41) | ![image](https://github.com/user-attachments/assets/9cd0e7b5-b54b-4c82-ab00-38ca9ee7d00d)


- 서로의 작업과 전혀 관련 없는 부분, 같은 코드를 다르게 수정한 부분 혼재
- 일일이 수작업으로 합쳐야함
- 때로는 서로의 코드를 합치다 실수가 생길수도

**[예시2]**

C 회사가 제작한 프로그램을 여러 회사에 납품한다.

![image](https://github.com/user-attachments/assets/022714a6-569d-4f5d-956d-cfedb051652a)

프로그램을 사용하는 회사들은 C 회사에 각자의 요구사항을 전달한다. C 회사는 각 회사의 요구 사항의 수만큼 코드를 복사하고, 수정해야 한다. 

![image](https://github.com/user-attachments/assets/bdea1642-99cf-416e-9c84-dea9257498b8)

시간이 흘러 프로그램은 v10.0.0까지 출시했고, 요구 사항은 1000개가 넘어갈 정도로 많아졌다. (v1.0.0에 대한 요구 사항 30개, v.3.0.0에 대한 요구 사항 80개 등) 버전별로 복사본을 만들고, 요구 사항별로 복사본을 만들어 요구 사항에 맞게 수정해야 할까? 매우 번거로운 일이다.

![image](https://github.com/user-attachments/assets/ff5175b4-b13b-43d1-8900-22195fb3e264)


### 브랜치로 문제 해결하기

브랜치는 버전의 분기

브랜치로 버전의 분기를 관리하는 방법

![image](https://github.com/user-attachments/assets/5fd30c62-5cf9-4cdf-8e33-b8cfd4e8d57c)

1. 브랜치를 <span style="color:red"> 나눈다 </span>
2. 각자의 브랜치에서 <span style="color:red"> 작업한다 </span>
3. (필요하다면) 나눈 브랜치를 <span style="color:red"> 합친다 </span>


**[예시1 문제 해결]** 

- 브랜치를 나눈다.

![image](https://github.com/user-attachments/assets/6b0e52aa-4084-4013-8698-f6ca4aba0724)

- 각자의 브랜치에서 작업한다.

![image](https://github.com/user-attachments/assets/72bec97e-ac88-4cf2-94e5-dba9b5c8d52d)

- 나눈 브랜치를 합친다.

![image](https://github.com/user-attachments/assets/a823d858-ff06-4cf6-9042-ccf0fa72577f)

내가 작업한 영역에서 다른 사람이 수정한 게 있는지만 확인하면 된다. 다른 사람이 작업한 코드 중에서 내 영역이 아니면 굳이 볼 필요가 없다.

→ 이제 <span style="color:red"> 같은 부분을 다르게 수정한 부분 </span>만 보면 된다(깃이 알려준다!).


**[예시2 문제 해결]** 

![image](https://github.com/user-attachments/assets/a045b4dd-69ee-497b-b40a-9363f57b9352)


버전 별로 출시하고, 각 버전에 대한 요구 사항이 생기면 그때그때 브랜치를 나눠 수정하면 된다. 복사본을 일일이 만드는 것 보다 메모리도 적게 들고, 실수도 줄일 수 있고, 작업도 훨씬 간편해진다.


## [이론] 브랜치 나누기(24.10.06)

브랜치 이름 - 최초의 브랜치, master 브랜치(github에서는 main 브랜치라고 한다)

가장 기본적인, 최초의 브랜치이다. 지금까지 만든 커밋들은 모두 기본적으로 master 브랜치에 속해 있다(내 블로그 빼고!).


예를 들어 이 그림을 보자. 

![image](https://github.com/user-attachments/assets/5ac5c824-d4d4-4e88-b3a7-f327529c7f3d)

master 브랜치에는 커밋이 <span style="color:red"> 세 개 </span>, foo 브랜치에는 커밋이 <span style="color:red"> 다섯 개 </span> 쌓여 있다.

![image](https://github.com/user-attachments/assets/18352b97-9dfb-47b8-9314-8af35de2a616)

master 브랜치에는 커밋이 <span style="color:red"> 세 개 </span>, foo 브랜치에는 커밋이 <span style="color:red"> 다섯 개 </span>, bar 브랜치에는 커밋이 <span style="color:red"> 여섯 개 </span> 쌓여 있다.


### 브랜치 이름을 어떻게 지을까? 

![image](https://github.com/user-attachments/assets/d72cf3a0-cbbb-4624-9140-b4814aadba46)

강제하진 않지만 막 짓는 것보다 목적에 맞게 브랜치 이름을 설정하는 게 좋다.


&nbsp;

### 특정 브랜치에서 작업하기 : HEAD와 체크아웃
&nbsp;

<span style="background-color:#fff5b1"> HEAD </span>
- 현재 작업 중인 브랜치의 커밋을 가리킨다.
- 일반적으로 현재 작업 중인 브랜치의 <span style="color:red"> 최신 커밋 </span>을 가리킨다.
- 한 마디로 "내가 지금 어디에서 작업 중인가"를 가리킨다.

![image](https://github.com/user-attachments/assets/51ae9af8-b3bf-4671-af69-c55b8b6b99c4)

(연결리스트가 떠오른다...)

HEAD가 bar 브랜치의 6번 커밋을 가리키고 있다 ➡️ 현재 여섯 개의 커밋이 쌓여 있고, 최신 커밋을 bar 6번 커밋이다.

HEAD를 요리조리 옮겨서 작업 브랜치를 바꿀 수 있는데, 이 걸 체크 아웃이라고 한다.

<span style="background-color:#fff5b1"> 체크 아웃(checkout) </span>
- 특정 브랜치에서 작업할 수 있도록 환경을 바꾸는 것이다.
- HEAD의 위치를 특정 브랜치의 최신 커밋으로 옮긴다.


## [Windows 실습] 브랜치 나누기(24.10.07)

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

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

---

출처 [인프런 강민철님 모두의 깃&깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)