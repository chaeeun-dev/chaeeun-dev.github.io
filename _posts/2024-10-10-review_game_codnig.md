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

## 💡기본 템플릿(Win API) 분석

### 📌HDC, HWND

HDC, HWND같이 H가 붙은 키워드는 int, 즉 정수형으로 윈도우 내부에서 어떻게 사용되는지 식별하기 위해 붙은 이름표이다.

---

### 📌콜백함수(callback function)

콜백함수는 함수의 생성 시점과 호출 시점이 달라 일반 함수와 다른 호출 방법을 갖는 함수이다. 일반 함수는 호출 즉시 실행되지만, 콜백 함수는 미리 등록해 두고 추후에 호출할 수도 있고 바로 호출할 수도 있다. 

(코드로 구현한 부분은 제대로 이해하지 못해서 지금은 개념만 짚고 넘어가기로..)

참고하면 좋은 블로그 [[C++] 콜백 함수(Callback Function)](https://choi-dan-di.github.io/cpp/callback-function/)


---

### 📌lParam

![image](https://github.com/user-attachments/assets/3cc38a8e-aeae-47f6-bcf3-f2f5fe57c56c)

WM_MOUSEMOVE에서 마우스 좌표를 lParam에 저장하는데, x 좌표는 아래 4바이트, y 좌표는 위 4바이트에 저장된다.

---

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

---

&nbsp;

## 💡프레임워크 제작

### 📌int 자료형

✅int 자료형

| Type | Min value | Max value |
| --- | --- | --- |
| int8 | -128 | 127 |
| int16 | -32768 | 32767 |
| int32 | -2147483648 | 2147483647 |
| int64 | -2^63 | 2^63 - 1 |


✅uint 자료형

| Type | Min value | Max value |
| --- | --- | --- |
| uint8 | 0 | 255 |
| uint16 | 0 | 65535 |
| uint32 | 0 | 4294967295 |
| uint64 | 0 | 2^64 - 1 |


참고 사이트 [Data types](https://doc.embedded-wizard.de/uint-type?v=13.00)

---

### 📌미리 컴파일된 헤더

- pch 파일을 생성하고 자주 컴파일하는 헤더를 include한다.
- 프로젝트 속성에서 '미리 컴파일 된 헤더 파일'에 'pch.h'를 입력하고 '사용(/Yu)'으로 수정한다.
- pch.cpp 속성에서 '미리 컴파일 된 헤더'에 '만들기(/Yc)'로 수정한다.
- 여기서 윈도우 템플릿 파일에 헤더를 <span style="color:red"> 맨 위에 </span> 작성해야 한다!

(이거 안 해서 오류를 겪었기에.. 오류 게시글에도 정리해뒀었다.)

---

### 📌캐스팅 4총사

**캐스팅(형변환)이란?** 

하나의 자료형을 다른 자료형으로 변환하는 것이다. C 스타일 캐스팅은 의도가 불분명하여 에러가 발생할 수 있고, 의도와 다르게 결과가 나올 수 있다. 따라서 C++은 용도에 따라 사용하도록 4가지 캐스팅으로 나누어 도입하였고 C 스타일보다 안전하게 캐스팅할 수 있게 되었다.

4총사 중에서 static_cast와 dynamic_cast가 핵심이지만 4개 다 중요하다.
1. static_cast
2. dynamic_cast
3. const_cast
4. reinterpret_cast

&nbsp;

**static_cast** : 타입 원칙에 비춰볼 때 상식적인 캐스팅만 허용해준다.

- int -> float

```cpp
    int hp = 100;
    int maxHp = 200;

    float ratio = hp / maxHp;   // 결과 0, 오른쪽 '/' 연산부터 처리
    float ratio = (float)hp / maxHp;    // 결과 0.5, 캐스팅 처리 완료
```

- Player* -> Knight(Knight는 Player를 상속받음)

```cpp
    Knight* k = new Knight();
    Player* p = k;

    Knight* k2 = p; // 위험한 작업이니 에러 발생함
    Knight* k2 = (Knight*)p;    // 뭘 하는지 알고 괜찮으니 통과해줘
```


---

&nbsp;

&nbsp;