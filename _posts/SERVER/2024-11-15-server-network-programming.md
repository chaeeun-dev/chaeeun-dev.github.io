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

- Server에서 receive하지 않고, Client에서 Send한다면? -> 잘 작동한다.. 이유는 Client가 Server로 데이터를 보내는 게 아닌 내부의 SendBuffer에 데이터를 보낸 게 성공으로 보기 때문이다. 그 다음에 OS가 SendBuffer에 있는 데이터를 Server의 RecvBuffer에 들어가게 된다.

![image](https://github.com/user-attachments/assets/cd73b4c0-02ca-40ec-8466-b3fc9beba028)

---

### 오류 발생 및 해결

[오류]

오류가 277개나 발생했다. 작성한 코드에서 발생한 게 아니라 include한 헤더 파일에서 오류가 발생한다. (내가 작성한 것도 코드도 아닌데 대체 왜 그러는 거야!!!!!)

- C2065 선언되지 않은 식별자입니다.
- C2375 재정의. 연결이 다릅니다.

![image](https://github.com/user-attachments/assets/8068e4dd-19b2-466c-b86b-7d2dd5005e11)

[오류 해결 과정]
- pch.h 파일에 **#pragma once**가 있는 걸 확인했으니 이는 pass.
- ServerCore 프로젝트의 CorePch.h 파일에 ```#include <stdio.h>```가 누락 되어 있었다. 해결 완료! 그리고 순서에 따라서 오류가 발생할 수도 있다는 점 꼭 유의하기.

![image](https://github.com/user-attachments/assets/a7cc0057-948e-469f-9db8-e2418a289cca)

---

## TCP vs UDP(24.11.15)
앞의 소켓 프로그래밍 입문 수업의 코드는 TCP 코드였다.

TCP, UDP 개념은 기본 상식 + 면접 단골 질문이니 꼭 숙지할 것! 서버 쪽 아니더라도 꼭 알아야 한다.

---

### 네트워크 통신 개념 설명
네트워크 통신은 택배를 배송하는 것과 비슷하다.

- 한 아파트 내부에서 각자의 호 수나 닉네임으로 불러 택배를 보낼 수 있다.

![image](https://github.com/user-attachments/assets/9bdee7f4-ecd0-4f5b-9a75-6e1a10d18be5) 

![image](https://github.com/user-attachments/assets/0d657a03-3006-43d9-8eac-48139e136a6f)

- 다른 아파트로 택배를 보낼 때는 여러 단계를 거쳐서 택배를 보내야 한다.

![image](https://github.com/user-attachments/assets/89477405-9705-4346-bee7-70d1febebc71)

&nbsp;

실제 네트워크도 이와 마찬가지이다. 집은 단말기(컴퓨터, 휴대폰 등), 경비실은 스위치, 택배 배송 센터는 라우터로 볼 수 있다.

- 같은 네트워크 망에서 보낼 때

![image](https://github.com/user-attachments/assets/7f304a60-366e-4d97-abbc-3fa20d92c1b5)

- 다른 네트워크 망으로 보낼 때

![image](https://github.com/user-attachments/assets/060b7853-0b70-4a97-9e36-2e52d4ab5b21)

---

### 통신 모델
- 택배 예시 - 택배에 이런 정보들이 다 기입되어 보내진다.

![image](https://github.com/user-attachments/assets/1513b613-18c6-4b91-8d51-3202afc5bc0c)

- 네트워크 통신

![image](https://github.com/user-attachments/assets/d7831b3e-099a-45c1-8e90-cff14a37f64c)

- 정보를 기입하는 이유? 오류가 발생했을 때 책임을 명확하게 한다.

&nbsp;

- TCP IP 모델 - 알아야 되는 것은 '여러 단계로 정책을 정하고 있고, TCP는 트랜스포트(배송 정책)에 해당한다.'

![image](https://github.com/user-attachments/assets/4e5f26fb-5f2e-43cc-9c9c-e46e029ea693)

---

### TCP vs UDP 
**[차이점]**
- TCP 
1. 안전한 트럭
2. 전화 연결 방식

- UDP
1. 위험한 총알 배송
2. 이메일 전송 방식

&nbsp;

**[연결 지향성]**
- TCP - 연결형 서비스
1. 연결을 위해 할당되는 논리적인 경로가 있다.
2. 전송 순서가 보장된다.
(소켓 프로그래밍 입문 수업에서 다룬 코드를 보면, 서버는 클라가 올 때까지(accept) 계속 기다리는 것으로 TCP라는 걸 알 수 있다!)

- UDP - 비연결형 서비스
1. 연결이라는 개념이 없다.
2. 전송 순서가 보장되지 않는다.
3. 경계(Boundary)의 개념이 있다.

&nbsp;

**[속도와 신뢰성]**
- TCP - 신뢰성 Good, 속도 Bad
1. 분실이 일어나면 책임지고 다시 전송한다. - 신뢰성 Good
2. 물건을 주고 받을 상황이 아니면 일부만 보낸다 . - 흐름/혼잡 제어
3. 고려할 것이 많으니 속도가 느리다.

- UDP - 신뢰성 Bad, 속도 Good
1. 분실에 대한 책임이 없다. - 신뢰성 Bad
2. 일단 보내고 생각한다.
3. 단순하기 때문에 속도가 빠르다.

&nbsp;

**[데이터 경계 Boundary]**
- TCP
1. 경계의 개념이 없다. - 순서대로 도착하지만, 데이터가 쪼개질 수도 있다.

- UDP
1. 경계의 개념이 있다. - 순서는 바뀔 수도 있지만, 데이터가 온전한 상태로 도착한다.

![image](https://github.com/user-attachments/assets/892bfb1f-340f-40ab-aea1-63a04378d92f)

---

### 게임에서 TCP vs UDP?
게임에서는 TCP vs UDP? 양쪽이 다 쓰이는데, 어떤 것을 사용할 때 장단점을 명확히 알아야 한다. 리스크를 감당해서 발생하는 문제점을 처리할 수 있어야 한다. 

- TCP - 속도가 느리고 데이터가 끊길 수도 있지만, 전송 순서가 보장되고 분실에 대한 책임이 있다. (MMO는 대부분 TCP 사용함)
- UDP - 속도가 빠르고 완전한 데이터가 배송되지만, 전송 순서가 뒤죽박죽이고 데이터 분실이 발생할 수 있다. (속도가 빠른 FPS에서 사용할 수 있는데, 데이터 분실에 대한 처리 작업이 필요함)

---

### 게임 서버 vs 웹 서버
- 게임 서버 - Stateful, 서버와 접속 되어서 통신한다.
- 웹 서버 - Stateless, 요청이 오면 바로 반납하고 연결이 끊긴다.

---

### 소스 코드
- [Dummy client의 UDP 코드](https://github.com/chaeeun-dev/Server/blob/main/DummyClient/UDP.cpp)
- [Server의 UDP 코드](https://github.com/chaeeun-dev/Server/blob/main/Server/UDP.cpp)

---

## 논블로킹 소켓(24.11.21)

### 소켓 옵션
1. level (SOL_SOCKET, IPPROTO_IP, IPPROTO_TCP)
2. optname
3. optval

```cpp
	// SO_KEEPALIVE (주기적으로 연결 상태 확인)
	bool enable = true;
	::setsockopt(listenSocket, SOL_SOCKET, SO_KEEPALIVE, (char*)&enable, sizeof(enable));


	// SO_LINGER = 지연하다
	// SO_SNDBUF
	// SO_RCVBUF

	int32 sendBufferSize;
	int32 optionLen = sizeof(sendBufferSize);
	::getsockopt(listenSocket, SOL_SOCKET, SO_SNDBUF, (char*)&sendBufferSize, &optionLen);
	std::cout << "송신 버퍼 크기" << sendBufferSize << std::endl;

	//

	// SO_REUSEADDR - 주소 재사용
	{
		bool enable = true;
		::setsockopt(listenSocket, SOL_SOCKET, SO_REUSEADDR, (char*)&enable, sizeof(enable));
	}

	// IPPROTO_TCP
	// TCP_NODELAY = Nagle 알고리즘(*중요) 작동 여부 - 데이터가 충분히 크면 보내고, 작으면 보내지 않고 기다림.
	// 게임에서 필요한가? - 데이터가 작은데 꼭 보내야할 경우는 끄는 게 좋음.
```

---

### SocketUtils
SocketUtils에 나만의 라이브러리를 만들어 굳이 외우지 않고 함수를 호출하도록 만든다. 

---

### 논블로킹 소켓
- 논블로킹 소켓 : 게임에서 다수의 접속자를 모두 받아주는 건 매우 버겁다. 할 일이 없는 이용자는 빠르게 스킵하는 게 논블로킹 소켓이다.

- 블로킹 : 원하는 행위가 일어날 때까지 기다림
- 논블로킹 : 바로 빠져나오는데 한 번 더 체크를 해서 기다리는 이유를 판별해야 함. (문제가 일어난 것인지, 데이터가 잘 전달된 것인지 등) 

### 소스 코드
- [Dummy client의 NonblockingSocket 코드](https://github.com/chaeeun-dev/Server/blob/main/DummyClient/NonblockingSocket.cpp)
- [Server 의 NonblockingSocket 코드](https://github.com/chaeeun-dev/Server/blob/main/Server/NonblockingSocket.cpp)

---

## Selecet, WSAEventSelect(24.11.22)

### Select
알맞는 타이밍을 미리 예지해서 불필요한 행위를 하지 않도록 한다.

- socket set
1. 읽기[] 쓰기[] 예외[] 관찰 대상
2. select(readSet, writeSet, exceptSet); → 관찰 시작
3. 적어도 하나의 소켓 준비되면 리턴 → 낙오자는 알아서 제거
4. 남은 소켓 체크해서 진행

---

출처 [루키스님 게임 프로그래머 올인원 강의](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss)