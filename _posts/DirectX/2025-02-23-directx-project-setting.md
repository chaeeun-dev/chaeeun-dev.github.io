---
title: "[DirectX 12] #2-1. 프로젝트 설정"
excerpt: "DirectX 프로젝트 초기 설정"

categories:
  - DirectX12
tags:
  - [DirectX, ProjectSetting]

permalink: /DirectX12/project-setting/

toc: true
toc_sticky: true

date: 2025-02-23
last_modified_at: 2025-02-23
---

## 프로젝트 생성 및 초기 설정

- 비주얼 스튜디오에서 `Windows 데스크톱 애플리케이션` 프로젝트 생성
    - 프로젝트 이름: `Client`
    - 솔루션 이름: `Game`
    - 경로 지정 후 생성

---

## WinMain, 메시지 처리

- 기본적으로 `WinMain` 함수에서 `while` 루프를 돌면서 코드가 실행된다.
- 기본 코드에서는 `GetMessage` 함수 사용하는데, 이 함수는 입력이 없으면 대기 상태가 된다.
- 게임은 입력이 없어도 계속 실행(렌더링, AI 연산 등)되어야 하므로 `PeekMessage`로 변경한다.

---

## 미리 컴파일된 헤더(pch)

- pch(precompiled header)
    - 공용으로 (자주)사용하는 헤더 파일을 미리 컴파일하여 빌드 속도를 향상시킨다.
    - `stdafx.h`라는 이름으로도 사용된다.

- 설정 방법
    1. `Debug`, `x64` 설정
    2. 프로젝트 속성 → `모든 구성` & `모든 플랫폼`
    3. C/C++ → `미리 컴파일된 헤더 사용(/Yu)` → 파일 이름: `pch.h`
    4. `pch.cpp` 파일 속성 → `미리 컴파일된 헤더 만들기(/Yc)`

---

## 프로젝트 구조 및 설계

### 프로젝트 구조

- Client 프로젝트에 `Utils`, `Game` 폴더를 생성한다.
- `Game` 클래스 생성 후 `Init()`, `Update()` 함수를 작성한다.
- `WinMain` 함수에 Game 객체를 생성하고 실행한다.

---

### 게임 루프 구조

모든 게임은 매 프레임(Tick)마다 무한 루프를 돌면서 업데이트된다.

---

## 스마트 포인터

C++에서 `new` 키워드로 객체를 생성하면, `delete`를 직접 호출해야 한다. 안전한 메모리 관리를 위해 스마트 포인터를 사용한다.

```cpp
// 잘못된 예제 (delete 필요)
Game* game = new Game();  

// 올바른 예제 (자동 메모리 해제)
std::unique_ptr<Game> game = std::make_unique<Game>();  
```
Unreal 엔진에서도 스마트 포인터를 사용한다.

---

## 엔진 라이브러리 

### 라이브러리 프로젝트의 필요성

공용으로 사용할 기능을 별도의 프로젝트로 관리할 수 있다. 이를 활용하면 여러 프로젝트에서 공용으로 사용할 수 있어 유지보수에 유리하다.

---

### 정적 라이브러리 vs 동적 라이브러리

||**정적 라이브러리 (Static Library)**|**동적 라이브러리 (DLL)**|
|---|---|---|
|**파일 확장자**|`.lib`|`.dll`|
|**링크 방식**|컴파일 시 포함|실행 시 로드|
|**장점**|실행 속도 빠름, 배포 용이|여러 프로그램에서 공유 가능|
|**단점**|실행 파일 크기 증가|DLL이 없으면 실행 불가|

프로젝트에서는 정적 라이브러리를 사용한다.

---

### Engine 프로젝트 생성 및 설정

1. 새 프로젝트 추가 → `정적 라이브러리` 선택
2. 프로젝트 이름: `Engine`
3. 빌드 후 생성된 `Engine.lib`를 Client 프로젝트에서 사용할 예정
4. `EnginePch.h` 파일 추가
5. `d3dx12.h` 추가 (DirectX 12 관련 보조 라이브러리)

---

### 생성되는 파일 정리

기본적으로 빌드하면 `Debug`, `x64` 등 여러 폴더에 파일이 분산된다. 한 폴더에 정리하기 위해 프로젝트의 파일 탐색기에 `Output` 폴더를 생성한다.

Engine, Client 프로젝트 모두 설정하기
1. 프로젝트 속성
2. 구성 → `모든 구성`, `모든 플랫폼`
3. 일반 → 출력 디렉터리 → `$(SolutionDir)Output\`
4. 빌드 하면 생성 파일이 모두 `Output` 폴더에 저장된다.

---

## Engine 사용하기

Client 프로젝트에서 Engine의 기능을 사용하는 것이 최종 목표다!

---

### Engine 라이브러리 포함

- 헤더 및 라이브러리 경로 설정
    1. Client 프로젝트 속성 → `VC++ 디렉터리`
    2. 포함 디렉터리 추가 `$(SolutionDir)Engine\`
    3. 라이브러리 디렉터리 추가: `$(SolutionDir)Output\`

- 링커 설정 (2가지 방식)
    1. Client 프로젝트 속성 → 링커 → 입력 → 추가 종속성 → `Engine.lib`
    2. `pch.h` 파일에 `#pragma comment(lib, "Engine.lib")` 코드를 추가하면 1번 설정 없이도 자동으로 라이브러리를 포함할 수 있다.

---

### 프로젝트 연결 테스트

Clinent와 Engine 프로젝트의 연결 확인을 위해, Clinent 프로젝트에서 Engine 프로젝트의 함수 `HelloEngine()`을 호출해본다.

함수가 정상적으로 호출이 됐으면 기본적인 프로젝트 설정이 끝났다!

---

[프로젝트 설정1 커밋](https://github.com/chaeeun-dev/DirectX12/commit/00243bc52c36caa39e21133eeee6bb2cf6ea16c0)

[프로젝트 설정2 커밋](https://github.com/chaeeun-dev/DirectX12/commit/38c9f53816ae67553245569116bffb294686d3c7)

---

*출처* 
*[인프런 Rookiss님 강의](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*