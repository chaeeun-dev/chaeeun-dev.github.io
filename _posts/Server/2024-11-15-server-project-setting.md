---
title: "[Server] #15-11. 프로젝트 설정"
excerpt: "서버 구현을 위한 프로젝트 생성하고 설정하기"

categories:
  - Server
tags:
  - [Server, ServerSetting, Setting]

permalink: /theory/server/project-setting/

toc: true
toc_sticky: true

date: 2024-11-15
last_modified_at: 2025-03-20
---

## 들어가며

게임 서버 개발에서 엔진 코드와 서버 코드를 분리하는 것이 일반적이다. 이번 수업에서는 엔진과 서버, 클라이언트 프로젝트를 한 솔루션 안에 생성하고 필요한 설정을 해본다.

---

## 새 프로젝트 만들기

1. 새 프로젝트 생성
    - 콘솔 앱: `DummyClient` (클라이언트)
    - 정적 라이브러리: `ServerCore` (서버의 공통 라이브러리)
2. pch(Precompiled Header) 설정
    - pch를 사용하여 빌드 속도를 향상시킨다.
    - 프로젝트 속성에서 pch 사용으로 변경

![image](https://github.com/user-attachments/assets/79fca8f4-c67b-4875-8975-a501628c0abb)

세 개의 프로젝트가 생성됐다.
- `ServerCore` → 서버 라이브러리
- `Server` → 실제 서버 어플리케이션
- `DummyClient` → 클라이언트 프로그램

---

## 폴더 정리

- Binaries: 빌드된 실행 파일(`Server`, `DummyClient`)이 저장된다.
- Intermediate: 빌드 중 생성되는 중간 파일이 저장된다.
- Libraries: 외부 라이브러리를 저장한다. 

---

## 경로 설정

- `ServerCore` 프로젝트 설정
    - 출력 디렉터리 설정
        - 프로젝트 설정 → 출력 디렉터리: `$(SolutionDir)Libraries\ServerCore\$(Configuration)\` 
        - 빌드하면 `Libraries/ServerCore/Debug`나 `Libraries/ServerCore/Release`/에 라이브러리가 생성된다.

![image](https://github.com/user-attachments/assets/ef0134f1-7828-4476-b5f1-81b812ba47e1)

&nbsp;

- `Server` 프로젝트 설정
    - 출력 디렉터리 설정
        - 프로젝트 속성 → 출력 디렉터리: `$(SolutionDir)Binaries\$(Platform)\`
    - 헤더 파일 위치 추가
        - 프로젝트 속성 → C/C++ → 일반 → 추가 포함 디렉터리: `$(SolutionDir)ServerCore\`
    - 라이브러리 위치 추가
        - 프로젝트 속성 → 링커 → 일반 → 추가 라이브러리 디렉터리: `$(SolutionDir)Libraries\ServerCore\`
    - 정적 라이브러리 링크 설정
        - `pch.h` 파일에 아래 코드 추가
        ```cpp
        #ifdef _DEBUG   // 디버그 모드일 때
        #pragma comment(lib, "Debug\\ServerCore.lib")
        #else          // 릴리즈 모드일 때
        #pragma comment(lib, "Release\\ServerCore.lib")
        #endif
        ```

이 설정을 통해 `ServerCore.lib`를 `Server`에서 자동으로 링크할 수 있다. 

&nbsp;

- `ServerCore` 프로젝트 추가 작업
    - Thread 폴더에 `ThreadManager` 추가
    - TLS(Thread Local Storage) 개념 이해하기

> 💡 TLS란?
>
> - 코드 (X), 스택 (X), 힙 (O), 데이터 (O), TLS (X) - O: 조심해야하는 영역, X: 안심해도 되는 영역
> - 일반 변수를 사용하면 모든 스레드가 접근 가능하지만, `Thread_local` 키워드를 사용하면 각 스레드에서만 접근할 수 있는 변수를 생성할 수 있다. 
> - 각 스레드마다 독립적인 변수 공간을 만들기 위해 `thread_local`을 사용할 수 있다.

---

[Server 깃허브 저장소](https://github.com/chaeeun-dev/Server)

---

## 마치며

이번 시간에 배운 것
- 프로젝트를 `DummyClient`, `ServerCore`, `Server`로 분리한다.
- 폴더의 구조를 `Binaries`, `Intermediate`, `Libraries`로 정리한다.
- 각 프로젝트의 빌드 경로를 설정한다.
- 정적 라이브러리(`ServerCore.lib`)를 `Server`에서 자동으로 링크한다.
- TLS 개념을 이해하고 `thread_local`를 사용해 스레드별 데이터를 관리할 수 있다.
- → 이제 Server와 Client를 독립적으로 개발하면서, `ServerCore`를 공통 라이브러리로 사용할 수 있다.

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*