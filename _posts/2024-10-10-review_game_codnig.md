---
title: "[C++] 프로젝트 복습 & 부족한 개념 바로 잡기"
date: 2024-10-10 12:50:00 +09:00 
categories: C++
author_profile: false
toc : true
toc_sticky: true
---

&nbsp;

프로젝트를 본격적으로 진행하기 전에 복습이 필요한 것 같아 직접 코드를 작성하고 부족한 개념을 복습하고자 한다. [Practice_GameCoding](https://github.com/chaeeun-dev/Practice_GameCoding) 이 레파지토리에서 연습하고 이 포스트에 글을 정리할 것이다.😎


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

<span style="background-color:#fff5b1"> **reinterpret_cast_cast** </span> : 위험하고 가장 강력한 형태의 캐스팅이다. 포인터 -> 전혀 관계 없는 다른 타입으로 변환할 때 자주 사용된다(이 것도 거의 안 쓰는 문법).

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

&nbsp;

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

&nbsp;

✅전방 선언(forward declaration) : 프로그래머가 아직 완전한 정의를 제공하지 않은 식별자(유형, 변수 상수, 함수 등)의 선언이다. 컴파일러는 식별자의 특정 속성(메모리 할당 위한 크기, 자료형 등)를 알아야 하지만 식별자가 보유하는 특정 값(변수, 상수 등) 또는 정의(함수의 경우)와 같은 상세한 부분을 알 필요가 없다.
[위키백과 전방 선언](https://ko.wikipedia.org/wiki/%EC%A0%84%EB%B0%A9_%EC%84%A0%EC%96%B8#:~:text=%EC%A0%84%EB%B0%A9%20%EC%84%A0%EC%96%B8(forward%20declaration)%EC%9D%80,%EB%A5%BC%20%EB%82%98%ED%83%80%EB%83%84)

특정 클래스에서 다른 클래스를 포인터로 사용할 때, 미리 해당 클래스가 필요하다는 것을 알려준다. 클래스의 헤더 파일을 include 하지 않아도 되기 때문에 불필요한 include와 컴파일 속도 저하를 방지할 수 있다. [클래스 전방 선언](https://velog.io/@taeil314/C-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A0%84%EB%B0%A9-%EC%84%A0%EC%96%B8)

상속 받으면 전방선언 의미 없으니, 상속 받는 객체를 받으면 include 해줘야 한다. 예를 들어 부모 클래스 Scene을 상속 받은 자식 클래스 GameScene에서 Scene을 사용하고 싶을 땐 전방선언이 의미 없으니 헤더를 include 해야 한다.

&nbsp;

---

## 💡더블 버퍼링

&nbsp;

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

&nbsp;

스페이스바를 누르면 Player 객체에서 미사일이 발사되는 코드를 구현할 때, 미사일을 어디서 관리하는 게 좋을까? 정답은 없다. 보통 Scene or ObjectManager에서 관리하는데, Uniy와 Unreal 엔진에서는 Scene에서 Object를 관리한다.

여기선 ObjectManager를 통해 구현한다.

&nbsp;

💥ObjectManager::Add(Object* object) - Object를 Add하는 함수

```cpp
    // 예외 처리 
    if (!object)
        return;

	auto findIt = std::find(_objects.begin(), _objects.end(), object);	// 같은 object가 있다면(== end를 리턴했다면) return 함
	if (findIt != _objects.end())
		return;

	_objects.push_back(object);
```
&nbsp;
✅std::find() 함수

```cpp
template <class InputIterator, class T>
InputIterator find(InputUterator first, InputIterator last, const T& val);
```
범위 안(first ~ last 전까지 == [first, last)) 원소들 중 val과 일치하는 첫 번째 원소를 가리키는 반복자를 리턴한다. 일치하는 원소를 못찾으면 last를 리턴한다.

[모두의 코드 C++ 레퍼런스 - find 함수](https://modoocode.com/261)

&nbsp;

💥ObjectManager::Remove(Object* object) - Object를 Remove하는 함수

```cpp
	if (!object)
		return;

	auto it = std::remove(_objects.begin(), _objects.end(), object);
	_objects.erase(it, _objects.end());
	
	delete object;	// 괜찮을까?
```
&nbsp;
✅std::remove() 함수 - <algorithm>에 정의됨.

```cpp
template <class ForwardIterator, class T>
ForwardIterator remove(ForwardIterator first, ForwardIterator last, const T& val);
```

범위 내 [first, last)의 원소들 중에서 val과 값이 일치하는 원소들을 제거한 범위로 변환한다. 이 때 이 함수는 해당 범위의 맨 끝을 가리키는 반복자를 리턴한다. 실제로 원소를 제거하는 게 아니라 지워야 하는 원소들을 컨테이너 맨 뒤로 보내는 것이다. 즉, 함수가 리턴하는 반복자부터 last까지 원소들이 val과 일치하는 원소로 대체된다. 원소를 지우기 위해서는 erase 함수를 호출해야 한다. 

예를 들어 a를 제거하고 싶다면, v.erase(std::remove(v.begin(), v.end(), a), v.end());

이 글이 정리가 잘 돼있다. [erase()와 remove() 함수의 차이](https://velog.io/@doorbals_512/C-erase%EC%99%80-remove-%ED%95%A8%EC%88%98%EC%9D%98-%EC%B0%A8%EC%9D%B4)

&nbsp;

💥ObjetManager::Clear() - 메모리 정리하는 함수

```cpp
	std::for_each(_objects.begin(), _objects.end(), [=](Object* obj) { delete obj; });
	_objects.clear();
```
&nbsp;
✅std::for_each() 함수 - <algorithm>에 정의됨.

```cpp
template <class InputIterator, class Function>
Function for_each(InputIterator first, InputIterator last, Function fn);
```

범위 내 [first, last) 원소들 각각에 대해 함수 fn을 실행한다. 참고로 함수의 리턴값은 무시된다. 

&nbsp;

💥 미사일을 소멸할 때 - 저 멀리 날아간 미사일을 소멸하고 싶을 때 ObjectManager의 Remove 함수를 호출한다. 이 때 <span style="color:red"> 끝나자마자 return 해야 </span> 한다. 메모리를 헤제했는데 사용하면 안 되기 때문!

&nbsp;

✅**댕글링 포인터(Dangling Pointer)** : 이미 해제된 메모리를 참조하거나 엑세스하려고 하는 상황으로, 예기치 않은 동작이나 프로그램 크래시, 데이터 손상 등의 문제를 일으킬 수 있다.

문제 발생하는 상황
1. 메모리 해제 후 포인터를 사용할 때
2. 스택 메모리에서 할당된 변수가 범위를 벗어났을 때 
3. 객체의 수명이 끝난 후에도 포인터를 계속사용할 때 

&nbsp;

---

## 💡리소스

라인 그려서 리소스 저장하고 파일 입출력하는 부분, 가볍게 훑고 패스.

&nbsp;

---

## 💡리소스 매니저

이 부분도 가볍게 훑고 넘어감.


&nbsp;

---

## 💡게임 수학

### 📌삼각함수

&nbsp;

각도 재는 방법
1. Degree 각도법(도수법) 0° ~ 360° 
2. Radian 각도법(호도법) 0 ~ 2π

게임에서의 삼각함수 활용 - 슈팅 게임에서 포신의 각도, 각도에 따라서 미사일이 어떻게 날아갈지(x 성분에는 cos(angle) 값 더해주고, y 성분에는 sin(angle) 값을 더해줌) 등

&nbsp;

### 📌벡터

&nbsp;

스칼라 vs 벡터
- 스칼라 : 고정, 변하지 않는 값.
- 벡터 : 크기와 방향을 가진 값. (목적지 - 시작점)

법칙
- 교환법칙 v1 + v2 = v2 + v1
- 결합법칙 (v1 + v2) + v3 = v1 + (v2 + v3)

벡터의 크기 구하기 - 피타고라스 개념으로 동일하게 구하면 됨.

정규화 : 단위 벡터로 만드는 것. 방향만 남김.

게임에서의 벡터? 미사일을 던질 때 유도탄 만들기

&nbsp;

✅미사일에서 포인터를 썼을 때 문제점

여러 개의 미사일을 한 번에 쐈을 때 하나의 미사일이 몬스터와 충돌하여 삭제되면 나머지 미사일은 타겟이 삭제 됐으니 원래 가던 방향으로 가버린다. 그래서 포인터를 생포인터로 넣지 말고

1. 스마트 포인터
2. 오브젝트 ID

이 방법으로 메모리 leak을 방지할 수 있다. 지금은 ID로 관리하는데 ObjectManager에서 ID로 객체를 구별하여 삭제하면 된다.


&nbsp;

struct로 Vector 자료형을 만들고, operator도 구현한다.

![image](https://github.com/user-attachments/assets/5903f9cb-58b7-40e5-97f8-28bb5b121769)

(코드가 길어서 앞에만 캡쳐했다.)

&nbsp;

### 📌내적(dot product)

&nbsp;

내적의 결과는 방향이 없는 스칼라 값이다.

```cpp
v1 = (x1, y1), v2 = (x2, y2)

v1⋅v2 = |v1||v2|cosθ = x1x2 + y1y2
```

&nbsp;

내적 응용
1. dot == 0 이면 직각
2. dot > 0 이면 θ < 90도
3. dot < 0 이면 θ > 90도
4. v1, v2 모두 단위벡터라면, 내적 결과는 cosθ
5. v1에 대한 v2의 성분을 구하려면(projection 투영)

&nbsp;

Vector 자료형에 내적 연산을 추가한다.

```cpp
	float Dot(Vector other)
	{
		return x * other.x + y * other.y;
	}
```

&nbsp;

### 📌외적(cross product)

&nbsp;

외적의 결과는 벡터이다. 외적은 순서가 바뀌면 방향이 반대로 바뀐다. 

```cpp
v1 = (x1, y1), v2 = (x2, y2)

v1 x v2 = (|v1||v2|sinθ)k = (x1y2 - x2y1)k
```

&nbsp;

외적 응용
1. 시계 / 반시계 방향 체크
2. 방향을 체크해 범위에서 벗어난 걸 체크

&nbsp;

Vector 자료형에 외적 연산을 추가한다.

```cpp
	float Cross(Vector other)
	{
		return x * other.y - y * other.x;
	}
```

&nbsp;

---

### 📌역삼각함수

&nbsp;

역함수는 항상 존재할까? NO! 1 대 1 대응 함수일 때만 존재한다.

삼각함수의 역함수 arccos, arcsin, arctan 등은 각도를 구할 때 사용한다(각도의 범위를 지정해야 함!). 

&nbsp;

---

### 📌속도와 가속도

&nbsp;

속도 = 거리 변화 / 시간

가속도 = 속도 변화 / 시간

---

## 💡포트리스 모작

포트리스 모작 부분은 가볍게 훑고 넘어갈 예정!(젤다 모작 부분을 다시 공부하려고 한 거기 때문)

### 📌Menu Scene -> Game Scene

&nbsp;

장면 전환하는 부분은 Menu Scene의 Update() 함수에서 InputManager를 불러와 특정 키가 눌렸는지 확인하면, SceneManager를 불러와 ChangeScene()을 호출하도록 구현한다.

&nbsp;

---

### 📌Player 방향 바뀌었을 때 그리기

&nbsp;

Update 함수에서 'A' 키가 눌리면 방향을 Dir::Left로 설정하고, 'D' 키가 눌리면 Dir::Right로 설정한다. 

Render 함수에서 방향을 체크한 후 방향에 따라 리소스를 그냥 그릴지, 반전해서 그릴지 if문으로 나눈다.


&nbsp;

---

### 📌UI는 어디서 관리해야 할까, UIManager?

UI에 데이터와 관련된 정보(Player의 스테미나, 사격 각도, 시간 등)를 묶어서 관리하면 안 된다. 규모가 커질수록 사양에 대한 문제가 커질 수 있다. 그래서 이중으로 관리하더라도 데이터는 꼭 분리해서 관리해야 한다. 

&nbsp;

---

## 💡2D 프레임워크 설계

### 📌obj 파일이란?

&nbsp;

오브젝트 파일(object file)은 컴파일이 끝나 기게어로 변환된 파일을 말한다. 오브젝트 파일의 확장자는 .o나 .obj가 된다. [TCP SCHOOL.com](https://www.tcpschool.com/cpp/cpp_intro_programming#google_vignette)

![image](https://github.com/user-attachments/assets/441cabd7-42ac-4c55-a243-0583e97ef1f4)

&nbsp;

---

### 📌폴더 정리

&nbsp;

Resources folder - Resource 관리
Binaries folder - 빌드 결과물
Intermediate - 중간에 생성되는 결과물

&nbsp;

솔루션 속성 - 일반 - 출력 디렉터리 - $(SolutionDir)Binaries$(Platform)\$(Configuration)\로 수정하여 출력물이 Binaries folder에 들어가도록 한다.

솔루션 속성 - 일반 - 중간 디렉터리 - 출력 디렉터리 내용을 복사하고 $(SolutionDir)Intermediate$(Platform)\$(Configuration)\로 수정한다.

&nbsp;

---

&nbsp;

## 💡스프라이트

### 📌Sprite

&nbsp;

Sprite : 2D 게임의 그래픽을 말한다. 캐릭터나 아이템, 배경 등의 이미지를 나타낸다.

&nbsp;

---

### 📌Object vs Resource

&nbsp;

Resource는 하나만 존재해서 공유해서 사용한다! 공공재 느낌.

&nbsp;

---

### 📌Texture

&nbsp;

이미지 파일을 텍스처라고 한다.

ResourceBase를 상속받아 Texture class를 생성한다.

&nbsp;

Transparent 함수 - 과거 bmp 파일에는 RGB 값이 들어있는데, 최근에는 RGB에 알파가 더해진 RGBA를 사용한다. 최근 리소스라면 transparent 함수는 사용하지 않아도 되는데 리소스가 과거 파일이라 알파값을 이 함수를 사용해 따로 넣어줘야 한다. 

&nbsp;

![image](https://github.com/user-attachments/assets/7903de9c-73dd-4686-b071-970ae607ca38)

이미지 로드해서 그리기 완료!

---

### 📌Sprite
 
&nbsp;

통채로 그려져 있는 텍스처를 잘라서 사용한다. class도 따로 생성하는데 Texture를 포인터로 받아서 잘라 쓸 것!(Texutre 클래스와 거의 비슷함)

![image](https://github.com/user-attachments/assets/02f98027-2c6b-475a-94bc-7cdaee2f1d26)

스프라이트로 잘라서 그리기 완료!

&nbsp;

---

&nbsp;

## 💡코드 구조 설계

### 📌Unreal Actor

&nbsp;

Actor : Scene에 배치할 수 있는 모든 Object를 뜻한다. 

Sprite Actor : 모든 Object는 Sprite가 필요한 게 아니다. Sprite가 필요한 액터만 Actor를 상속받아 Rendering될 수 있도록 한다.

&nbsp;

Unreal은 Init, Update, Render를 BeginPlay, Tick, Render로 표현한다.



--- 

&nbsp;

## ⛅날짜별 공부한 내용 & check-list

24.10.10 기본 템플릿 분석 ~ 프레임워크 제작의 캐스팅 4총사까지

24.10.11 프레임워크 제작의 캐스팅 4총사 ~ 매니저(타임 매니저까지)

24.10.12 프레임워크 제작의 매니저 ~ Object 설계의 Object 클래스까지

24.10.13 Object 설계의 Object 매니저(STL 함수 정리), 기존 프로젝트에 pch 파일 오류 났는데 아직 해결 못함!

24.10.14 pch 문제 해결 완료, 리소스 ~ 게임 수학의 외적까지

24.10.15 게임수학, 포트리스 모작 완료

24.10.16 2D 게임 프레임워크 설계의 스프라이트 2/3정도 완료(리소스 로드해서 화면 출력까지)

24.10.18 스프라이트 끝(로드한 텍스처를 잘라서 스프라이트 만들어서 화면 출력)

&nbsp;

check-list

10월 ~~10, 11, 12, 13, 14, 15, 16, 17, 18,~~ 19, 20, 21

- [x]  Windows API 입문
    - [x]  기본 템플릿 분석(24.10.10)
    - [x]  프레임워크 제작(24.10.10~12)
    - [x]  Scene과 SceneManager(24.10.12)
    - [x]  더블 버퍼링(24.10.12)
    - [x]  오브젝트 설계(24.10.12 ~ 13)
    - [x]  리소스(24.10.14)
    - [x]  리소스 매니저(24.10.14)
- [x]  게임 수학
    - [x]  삼각함수(24.10.14)
    - [x]  벡터(24.10.14)
    - [x]  내적(24.10.14)
    - [x]  외적(24.10.14)
    - [x]  역삼각함수(24.10.15)
    - [x]  포트리스 모작(24.10.15)
    - [x]  속도와 가속도(24.10.15)
- [ ]  2D 게임 프레임워크 설계
    - [x]  스프라이트(24.10.16, 18)
    - [ ]  코드 구조 설계
    - [ ]  애니메이션
    - [ ]  카메라
    - [ ]  충돌
    - [ ]  UI
- [ ]  2D 포폴 준비
    - [ ]  타일맵
    - [ ]  파일 입출력
    - [ ]  사운드
    - [ ]  충돌처리
    - [ ]  충돌 레이어
    - [ ]  State 패턴
    - [ ]  젤다 실습

&nbsp;

## ➕추가로 정리할 개념들

동적 바인딩, 싱글톤(메크로 함수에서), ~~전방선언~~, 스마트 포인터, vector&map&list 등 포스트 만들어서 따로 정리하기!!!!