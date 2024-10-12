---
title: "[C++] 프로젝트 복습 & 부족한 개념 바로 잡기"
date: 2024-10-10 12:50:00 +09:00 
categories: C++
author_profile: false
toc : true
toc_sticky: true
---

&nbsp;

## 💡기본 템플릿(Win API) 분석

### 📌HDC, HWND

&nbsp;

HDC(도화지 번호), HWND(윈도우 핸들 번호)같이 H가 붙은 키워드는 int, 즉 정수형으로 윈도우 내부에서 어떻게 사용되는지 식별하기 위해 붙은 이름표이다.

핸들이란? 운영체제 내부에 있는 어떤 리소스의 주소를 정수로 치환한 값.

---

### 📌콜백함수(callback function)

&nbsp;

콜백함수는 함수의 생성 시점과 호출 시점이 달라 일반 함수와 다른 호출 방법을 갖는 함수이다. 일반 함수는 호출 즉시 실행되지만, 콜백 함수는 미리 등록해 두고 추후에 호출할 수도 있고 바로 호출할 수도 있다. 

(코드로 구현한 부분은 제대로 이해하지 못해서 지금은 개념만 짚고 넘어가기로..)

참고하면 좋은 블로그 [[C++] 콜백 함수(Callback Function)](https://choi-dan-di.github.io/cpp/callback-function/)

&nbsp;

---

### 📌lParam

&nbsp;

![image](https://github.com/user-attachments/assets/3cc38a8e-aeae-47f6-bcf3-f2f5fe57c56c)

WM_MOUSEMOVE에서 마우스 좌표를 lParam에 저장하는데, x 좌표는 아래 4바이트, y 좌표는 위 4바이트에 저장된다.

&nbsp;

---

### 📌기존 main 루프의 문제점

&nbsp;

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

GetMessage 함수는 메시지를 큐에서 가져왔으면 큐에서 해당 메시지를 제거한다. PeekMessage 함수는 메시지를 확인하고 큐에서 제거하지 않는다. 

✅정리하자면, PeekMessage 함수는 프로그램이 끊임없이 돌아가야 하는 게임 상황에서 메세지를 처리할 때 사용하면 된다.

&nbsp;

---

&nbsp;

## 💡프레임워크 제작

### 📌int 자료형

&nbsp;

✅int 자료형

| Type | Min value | Max value |
| --- | --- | --- |
| int8 | -128 | 127 |
| int16 | -32768 | 32767 |
| int32 | -2147483648 | 2147483647 |
| int64 | -2^63 | 2^63 - 1 |

&nbsp;

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

&nbsp;

- pch 파일을 생성하고 자주 컴파일하는 헤더를 include한다.
- 프로젝트 속성에서 '미리 컴파일 된 헤더 파일'에 'pch.h'를 입력하고 '사용(/Yu)'으로 수정한다.
- pch.cpp 속성에서 '미리 컴파일 된 헤더'에 '만들기(/Yc)'로 수정한다.
- 여기서 윈도우 템플릿 파일에 헤더를 <span style="color:red"> 맨 위에 </span> 작성해야 한다!

(이거 안 해서 오류를 겪었기에.. 오류 게시글에도 정리해뒀었다.)

---

### 📌캐스팅 4총사

&nbsp;

**캐스팅(형변환)이란?** 

하나의 자료형을 다른 자료형으로 변환하는 것이다. C 스타일 캐스팅은 의도가 불분명하여(문법이 하나라서) 에러가 발생할 수 있고, 의도와 다르게 결과가 나올 수 있다. 따라서 C++은 용도에 따라 사용하도록 4가지 캐스팅으로 나누어 도입하였고 C 스타일보다 안전하게 캐스팅할 수 있게 됐다.

&nbsp;

4총사 중에서 static_cast와 dynamic_cast가 핵심이다(면접에서는 4개 다 나올 수 있음).
1. static_cast
2. dynamic_cast
3. const_cast
4. reinterpret_cast

&nbsp;

<span style="background-color:#fff5b1"> **static_cast** </span> : 타입 원칙에 비춰볼 때 상식적인 캐스팅만 허용해준다.

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
    Player* p = k;  // Knight를 Player로 변환할 때... Knight는 Player인가? 당연하다(에러 발생X)

    Knight* k2 = p; // 위에서 변환한 Player를 Knight로 원복... Player는 Knight인가? 위험한 작업이니 에러 발생함
    Knight* k2 = (Knight*)p;    // 뭘 하는지 알고 괜찮으니 통과해줘
```

Knight -> Player -> Knight 로 캐스팅 하는 건 괜찮은데,

Archer -> Player -> Knight 로 캐스팅한다면 위험하다. <span style="color:red"> memory leak </span>이 일어날 수 있다.

안전한 방법은 아니다. 위험하지만, 뭘 하는지 알고 있다고 안심시키는 것. 

&nbsp;

<span style="background-color:#fff5b1"> **dynamic_cast** </span> : 상속 관계에서의 안전 변환을 할 수 있다.. 다형성을 활용하는 방식이다(virtual).

virtual이 하나라도 있어야 활용할 수 있다. ➡️ RTTI(RunTime Type Information)

상위 클래스인 Player가 있고 이를 상속받은 Knight와 Archer 클래스가 있다.

```cpp
    class Player()
    {
    public:
        Player() {}
        virtual ~Player() {}  // virtual이 있어야 dynamic_cast할 수 있음
    };

    class Knight : public Player
    {
    public:
        Knight() {}
        virtual ~Knight() {}

    };

    class Archer : public Player
    {

    };
    
```

&nbsp;

✅virtual 함수

객체에 가상함수가 있으면 첫 번째 공간에 가상함수 테이블을 가리키는 공간이 할당 된다. 테이블에는 호출해야 하는 테이블 주소들이 기입된다. 테이블의 정보로 어떤 함수를 호출할지 판단할 수 있다(동적 바인딩의 원리).

![image](https://github.com/user-attachments/assets/607567a8-82f5-4e48-b186-ae32c834cc5e)

가상 함수가 필요한 이유는? 가상 함수 테이블에 있는 정보를 이용해 판별하는 것이기 때문(RTTI)!

&nbsp;

```cpp
    Archer* k = new Archer();
    Player* p = k;

    Knight* k2 = dynamic_cast<Knight*>(p);  // dynamic_cast가 캐스팅이 안 되도록 막아주니 k2 == 0
```

Archer -> Player -> Knight 로 캐스팅하려고 하면 dynamic_cast가 안 되게 막아준다.

캐스팅이 되는 경우(Knight -> Player -> Knight)에만 캐스팅해주고, 안 되는 경우(Archer -> Player -> Knight)에는 0을 넣어준다.

&nbsp;

✅static_cast vs dynamic_cast

Item을 Weapon으로 변환하고 싶을 때
```cpp
    Item* item = DropItem();

    // static_cast - ItemType이 Weapon인지 확인해야 함
    ItemType itemType = item->GetItemType();    
    if (itemType == IT_Weapon)
        Weapon* weapon = static_cast<Weapon>(item);

    // dynamic_cast - 위 코드처럼 확인하지 않고 한 번에 처리할 수 있음
    Weapon* weapon = static_cast<Weapon>(item);
```

&nbsp;

<span style="background-color:#fff5b1"> **const_cast** </span> : const 여부를 뗐다가 붙였다가 할 수 있다(거의 안 쓰는 문법).

```cpp
    const char* name = "Hello";
    char name2 = const_cast<char*>(name);   // 굳이..? 바꿀거면 const로 하지 말았어야.. 그래서 잘 안 쓰는 문법
```

&nbsp;

<span style="background-color:#fff5b1"> **reinterpret_cast_cast** </span> : 위험하고 가장 강력한 형태의 캐스팅이다. 포인터 -> 전혀 관계 없는 다른 타입으로 변환할 때 자주 사용된다(이 것도 거의 안 쓴느 문법).

Knight -> Dog 로 뜬금 없지만 캐스팅이 필요할 때, 위험한 거 알겠으니 캐스팅 해달라.

&nbsp;

---

### 📌99.Headers

&nbsp;

Headers 폴더
- Defines : 메크로 작성해서 편리하게 사용하도록
- Enums : Enum 관리 (예, SceneType, PlayerType 등)
- pch : 자주 사용하는 헤더 include
- Utils : 자주 사용하는 helper 함수 정리

&nbsp;

---

### 📌매니저

&nbsp;

매니저는 항상 singleton(singleton은 클래스의 instance를 하나로 제한할 때 사용)으로 작성한다. 매크로를 사용하는데, 매크로는 디버깅하기 번거롭다는 단점이 있어 필요한 부분에만 사용해야 한다.


- TimeManager : 시간, 프레임 관련 처리

```cpp
	uint64 currentCount = {};
	::QueryPerformanceCounter(reinterpret_cast<LARGE_INTEGER*>(&currentCount));

	_deltaTime = (currentCount - _prevCount) / static_cast<float>(_frequency);
	_prevCount = currentCount;

	_frameCount++;	// 몇 번 호출 됐는지 계속 추적
	_frameTime += _deltaTime;	// 경과된 시간 추적

	if (_frameTime >= 1.f)
	{
		// 1초동안 몇 번의 함수가 호출?
		_fps = static_cast<uint32>(_frameCount / _frameTime);	// 목적은 fps를 구하기 위한 것

		_frameTime = 0.f;
		_frameCount = 0;
	}

```

✅CPU 클럭 : 클럭 속도는 CPU가 초당 실행하는 사이클 수를 GHz(기가 헤르츠) 단위로 측정한 것이다. 클럭이 높다는 것은 1초에 더 많은 양의 데이터를 처라한다는 의미이다. 1Hz는 1개의 전기 신호, 1MHz는 100만 개의 전기 신호, 1GHz는 10억 개의 전기 신호를 나타낸다.

::QueryPerformanceCounter 함수를 통해 currentCount 변수에 현재 시스템의 정확한 시간 값을 저장한다.

_deltaTime = (currentCount - _prevCount) / static_cast<float>(_frequency);

현 시점의 카운터 값 - 이전 시점의 카운터 값) == 경과된 카운트

경과된 카운트 / 초당 카운터가 돌아가는 수 == 프레임이 시작된 이후 경과된 시간(deltaTime)

&nbsp;

- InputManager : 키보드, 마우스 입력 처리

&nbsp;

---

## 💡Scene과 SceneManager

### 📌Scene

&nbsp;

Scene은 화면의 모든 작업, 객체의 모든 처리 등을 담당한다. 최상위 클래스로 순수 가상 함수로 만들어 자식 클래스가 구현하도록 만든다.

```cpp
    virtual void Init() abstract;
    virtual void Update() abstract;
    virtual void Render(HDC hdc) abstract;
```

&nbsp;

SceneManager는 Scene을 관리할 매니저로, GameScene, DevScene 등을 관리해준다. 대표적으로 Scene을 change할 때 새로운 Scene을 생성하고, 이전 Scene을 삭제하는 일을 수행한다.

&nbsp;

---

### 📌전방선언

✅전방 선언(forward declaration) : 프로그래머가 아직 완전한 정의를 제공하지 않은 식별자(유형, 변수 상수, 함수 등)의 선언이다. 컴파일러는 식별자의 특정 속성(메모리 할당 위한 크기, 자료형 등)를 알아야 하지만 식별자가 보유하는 특정 값(변수, 상수 등) 또는 정의(함수의 경우)와 같은 상세한 부분을 알 필요가 없다.
[위키백과 전방 선언](https://ko.wikipedia.org/wiki/%EC%A0%84%EB%B0%A9_%EC%84%A0%EC%96%B8#:~:text=%EC%A0%84%EB%B0%A9%20%EC%84%A0%EC%96%B8(forward%20declaration)%EC%9D%80,%EB%A5%BC%20%EB%82%98%ED%83%80%EB%83%84)

특정 클래스에서 다른 클래스를 포인터로 사용할 때, 미리 해당 클래스가 필요하다는 것을 알려준다. 클래스의 헤더 파일을 include 하지 않아도 되기 때문에 불필요한 include와 컴파일 속도 저하를 방지할 수 있다. [클래스 전방 선언](https://velog.io/@taeil314/C-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A0%84%EB%B0%A9-%EC%84%A0%EC%96%B8)

상속 받으면 전방선언 의미 없으니, 상속 받는 객체를 받으면 include 해줘야 한다. 예를 들어 부모 클래스 Scene을 상속 받은 자식 클래스 GameScene에서 Scene을 사용하고 싶을 땐 전방선언이 의미 없으니 헤더를 include 해야 한다.

&nbsp;

---

## 💡더블 버퍼링

✅더블 버퍼링(dubble buffering) : 화면에 표시해야 할 내용을 미리 메모리에 그려 놓고 화면으로 한 번에 전송하는 방법이다. 

_hdcBack - 새로운 도화지, 기존의 도화지인 _hdc를 _hdcBack으로 바꾸어 이 곳에 그림을 그리도록 수정한다. _hdcBack에 그린 그림을 비트 블릿 함수(고속 복사)로 _hdc에 복사하면 끝!

&nbsp;

---

## 💡오브젝트 설계

### 📌Object 클래스

&nbsp;

Object를 최상위 클래스(가상 함수)로 만들어서 ObjectType(Player, Monster등)에 따라 상속받아 구현하도록 만든다. 

Player 클래스를 구현할 때, Init() 함수에 Stat을 초기화하도록 했는데, 원래 게임의 고정값은 데이터 시트를 받아서 관리해야 한다(프로젝트에서 엑셀로 관리해 받아올 예정임).

&nbsp;

---

### 📌Object 매니저


---

## 💡리소스



---

## 💡리소스 매니저




---

&nbsp;

24.10.10 기본 템플릿 분석 ~ 프레임워크 제작의 캐스팅 4총사까지

24.10.11 프레임워크 제작의 캐스팅 4총사 ~ 매니저(타임 매니저까지)

24.10.12 프레임워크 제작의 매니저 ~ Object 설계의 Object 클래스까지


&nbsp;

필요한 개념 정리 - 동적 바인딩, 싱글톤(메크로 함수에서), ~전방선언~ 