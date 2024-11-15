---
title: "[SERVER] 네트워크 프로그래밍"
date: 2024-11-15 12:50:00 +09:00 
categories: SERVER
author_profile: false
toc : true
toc_sticky: true
---

## 소켓 프로그래밍 입문(24.11.15)
함수가 너무 정신없이 등장하는데 처음에는 정말 어려울 수 있다. 구글링 많이 하면서 함수랑 친해지기!

&nbsp;

[폴더 설명]
- Server : 게임 서버
- DummyClient : 만들었던 2D 게임
- ServerCore : 네트워크 라이브러리

- 주의 : 프로젝트의 헤더 순서에 따라서 에러가 발생할 수도 있으니 순서 확인 잘 하자.

### 서버가 돌아가는 방법 - 식당 비유
1. 새로운 소켓 생성 <span style="color:red">socket</span> - 손님에게 직원 배치
2. 소켓에 주소/포트 번호 지정 <span style="color:red">bind</span> - 직원에게 교육 시키기
3. 소켓 일 시키기 <span style="color:red">listen</span> - 근무 시작
4. 손님 접속 <span style="color:red">accept</span> - 외부에서 손님 유입
5. 클라와 통신 - 정상적인 서비스 운영

### Server와 Client 코드 

- [커밋 링크](https://github.com/chaeeun-dev/Server/commit/4b8df03fdea3023b8980f92037fca3c07cc5ea7d#diff-b366230e1ece8ff7ba990db4f08b0c7ff73f50434bec6ea29bdb9629b1ade904)

- 프로젝트 동시에 실행하는 방법 : 프로젝트 속성 - 여러 개의 시작 프로젝트 선택 - 작업 시작 

![image](https://github.com/user-attachments/assets/01ed3fe8-5b22-47ec-af87-6bdb761c994a)

&nbsp;

- Server에서 receive하지 않고, Client에서 Send한다면? -> 잘 작동한다.. 이유는 Client가 Server로 데이터를 보내는 게 아닌 내부의 SendBuffer에 데이터를 보낸 게 성공으로 보기 때문이다. 그 디음에 OS가 SendBuffer에 있는 데이터를 Server의 RecvBuffer에 들어가게 된다.

![image](https://github.com/user-attachments/assets/cd73b4c0-02ca-40ec-8466-b3fc9beba028)

---

### 오류 발생 및 해결

오류가 277개나 발생했다. 작성한 코드에서 발생한 게 아니라 include한 헤더 파일에서 오류가 발생한다. (내가 작성한 것도 코드도 아닌데 대체 왜 그러는 거야!!!!!)

- C2065 선언되지 않은 식별자입니다.
- C2375 재정의. 연결이 다릅니다.

![image](https://github.com/user-attachments/assets/a7cc0057-948e-469f-9db8-e2418a289cca)

&nbsp;

[오류 해결 과정]
- pch.h 파일에 **#pragma once**가 있는 걸 확인했으니 이는 pass.
- ServerCore 프로젝트의 CorePch.h 파일에 ```#include <stdio.h>```가 누락 되어 있었다. 해결 완료! 그리고 순서에 따라서 오류가 발생할 수도 있다는 점 꼭 유의하기.

![image](https://github.com/user-attachments/assets/8068e4dd-19b2-466c-b86b-7d2dd5005e11)

---

출처 [루키스님 게임 프로그래머 올인원 강의](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss)