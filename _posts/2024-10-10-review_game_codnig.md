---
title: "[C++] 프로젝트 복습 & 부족한 개념 바로 잡기"
date: 2024-10-10 12:50:00 +09:00 
categories: C++
author_profile: false
toc : true
toc_sticky: true
---

&nbsp;

# 24.10.10(목)

## 기본 템플릿 분석

Win API 템플릿 분석 

### 📌HDC, HWND

HDC, HWND같이 H가 붙은 키워드는 int, 즉 정수형으로 윈도우 내부에서 어떻게 사용되는지 식별하기 위해 붙은 이름표이다.

&nbsp;

### 📌콜백함수(callback function)

콜백함수는 함수의 생성 시점과 호출 시점이 달라 일반 함수와 다른 호출 방법을 갖는 함수이다. 일반 함수는 호출 즉시 실행되지만, 콜백 함수는 미리 등록해 두고 추후에 호출할 수도 있고 바로 호출할 수도 있다. 

(코드로 구현한 부분은 제대로 이해하지 못해서 지금은 개념만 짚고 넘어가기로..)

참고하면 좋은 블로그 [[C++] 콜백 함수(Callback Function)](https://choi-dan-di.github.io/cpp/callback-function/)


&nbsp;

### 📌lParam

![image](https://github.com/user-attachments/assets/3cc38a8e-aeae-47f6-bcf3-f2f5fe57c56c)

WM_MOUSEMOVE에서 마우스 좌표를 lParam에 저장하는데, x 좌표는 아래 4바이트, y 좌표는 위 4바이트에 저장된다.

&nbsp;

### 📌기존 main 루프의 문제점

```cpp
    while (::GwtMwssage(&msg, nullptr, 0, 0))
    {
        ::TranslateMessage(&msg);
        ::DispatchMessage(&msg);
    }
```

게임은 <span style="color:red"> 입력, 로직, 렌더링 </span> 3단계가 무한으로 돌아가야 하는데, 기존 Win API의 GetMessage 함수는 사용자 입력이나 시스템 메시지가 들어올 때까지 대기한다. 

게임 프로그래밍 패턴 ch9 게임 루프 내용 - 게임은 유저 입력 없이도 계속 돌아간다. 아무것도 하지 않아도 게임 화면은 멈추지 않고 애니메이션과 시각적 연출을 계속 한다. ~ 루프에서 사용자 입력을 처리하지만 마냥 기다리고 있지 않다는 점, 이게 게임 루프의 핵심이다. 루프는 끊임 없이 돌아간다.

```cpp
    while (true)
    {
        Input();
        update();
        render();
    }
```

➡️GetMessage 함수 대신 PeekMessage 함수 사용하기

GetMessage 함수는 메시지 큐에서 메시지를 가져올 때 큐에 메시지가 없으면 메시지가 올 때까지 블로킹되어 다른 코드를 실행할 수 없다. PeekMessage 함수는 큐에 메시지가 없어도 블로킹되지 않고 즉시 반환되어 다른 작업을 수행하거나 대기 상태를 회피할 수 있다. 

GetMessage 함수는 메시지를 큐에서 가져왔으면 큐에서 해당 메시지를 제거한다. PeekMessage 함수는 메시지를 확인핟고 큐에서 제거하지 않는다. 

✅정리하자면, PeekMessage 함수는 프로그램이 끊임없이 돌아가야 하는 게임 상황에서 메세지를 처리할 때 사용하면 된다.

&nbsp;

## 프레임워크 제작
