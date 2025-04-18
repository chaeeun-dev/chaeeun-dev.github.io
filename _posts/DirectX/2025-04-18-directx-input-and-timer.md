---
title: "[DirectX 12] #3-1. Input과 Timer"
excerpt: "Input과 Timer 클래스 구현하기"

categories:
  - DirectX12
tags:
  - [DirectX, Input, Timer]

permalink: /DirectX12/input-and-timer/

toc: true
toc_sticky: true

date: 2025-04-18
last_modified_at: 2025-04-18
---

## 들어가며

게임에서는 키보드나 마우스의 입력과 시간에 따른 계산이 핵심이다. 이번 시간에는 카메라를 이동하거나 객체를 움직이기 위한 Input 처리, 그리고 프레임마다 시간값을 추적하는 Timer를 구현해 볼 것이다.

Input과 Timer는 싱글톤으로 구현하거나 엔진에서 객체로 관리할 수 있는데, 이번 수업에서는 Engine 클래스에서 객체로 관리하는 방식을 사용한다.

---

## Input 클래스

키보드 입력을 감지하여 `W`, `A`, `S`, `D` 등을 눌러 객체를 이동할 수 있도록 해보자. 특정 키가 눌림, 떨어짐, 유지 상태인지 추적한다.

&nbsp;

- Key 정의
    - Press: 누르고 있는 상태
    - Down: 처음 눌렀을 때
    - Up: 눌렀다 뗐을 때

```cpp
enum class KEY_TYPE
{
	UP = VK_UP, DOWN = VK_DOWN, LEFT = VK_LEFT, RIGHT = VK_RIGHT,
	W = 'W', A = 'A', S = 'S', D = 'D',
};

enum class KEY_STATE
{
	NONE, PRESS, DOWN, UP, END
};
```

&nbsp;

- `Input::Init()`

```cpp
_hwnd = hwnd;
_states.resize(KEY_TYPE_COUNT, KEY_STATE::NONE);
```

&nbsp;

- `Input::Update()`
    - 게임 창 포커스 상태에서만 키 입력을 받도록 한다. (핸들 확인)
    - `GetAsyncKeystate()`로 현재 키 상태를 매 프레임 가져와 상태를 갱신한다.

```cpp
HWND hwnd = ::GetActiveWindow();
	if (_hwnd != hwnd)
	{
		for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
			_states[key] = KEY_STATE::NONE;

		return;
	}

	for (uint32 key = 0; key < KEY_TYPE_COUNT; key++)
	{
		// 키가 눌려 있으면 true
		if (::GetAsyncKeyState(key) & 0x8000)
		{
			KEY_STATE& state = _states[key];

			// 이전 프레임에 키를 누른 상태라면 PRESS
			if (state == KEY_STATE::PRESS || state == KEY_STATE::DOWN)
				state = KEY_STATE::PRESS;
			else
				state = KEY_STATE::DOWN;
		}
		else
		{
			KEY_STATE& state = _states[key];

			// 이전 프레임에 키를 누른 상태라면 UP
			if (state == KEY_STATE::PRESS || state == KEY_STATE::DOWN)
				state = KEY_STATE::UP;
			else
				state = KEY_STATE::NONE;
		}
	}
```

&nbsp;

- 키 상태 확인 함수

```cpp
// 누르고 있을 때
bool GetButton(KEY_TYPE key) { return GetState(key) == KEY_STATE::PRESS; }
// 맨 처음 눌렀을 때
bool GetButtonDown(KEY_TYPE key) { return GetState(key) == KEY_STATE::DOWN; }
// 맨 처음 눌렀다 뗐을 때
bool GetButtonUp(KEY_TYPE key) { return GetState(key) == KEY_STATE::UP; }
```

---

## Timer 클래스

매 프레임마다 deltaTime(이전 프레임 대비 경과 시간)을 측정하고, 1초마다 FPS(Frame Per Second)를 계산한다.

&nbsp;

- 멤버 변수

```cpp
uint64 _frequency;   // 성능 카운터 빈도
uint64 _prevCount;   // 이전 프레임의 카운터
float  _deltaTime;

uint32 _frameCount;
float  _frameTime;
uint32 _fps;
```

&nbsp;

- `Timer::Init()`

```cpp
::QueryPerformanceFrequency(reinterpret_cast<LARGE_INTEGER*>(&_frequency));
::QueryPerformanceCounter(reinterpret_cast<LARGE_INTEGER*>(&_prevCount)); // CPU 클럭
```

&nbsp;

- `Timer::Update()`
    - deltaTime
        - `QueryPerformanceCounter()`는 CPU 클럭 기반의 정밀한 시간 측정을 한다.
        - `_frequency`는 1초 동안 카운트 수이고, 초기화시 `QueryPerformanceCounter()`로 받아온다.
        - 두 카운터의 차이를 빈도로 나누면 경과 시간(초)이 된다.
    - fps(초당 프레임 수)
        - `_frameCount`는 1초 동안 누적된 프레임 수이다.
        - `_frameTime`은 deltaTime을 누적해 1초가 될 때까지 시간을 계속 더한다.
        - 누적된 시간이 1초 이상 되면 `fps` 값을 갱신하고 초기화한다.

```cpp
	uint64 currentCount;
	::QueryPerformanceCounter(reinterpret_cast<LARGE_INTEGER*>(&currentCount));

	_deltaTime = (currentCount - _prevCount) / static_cast<float>(_frequency);
	_prevCount = currentCount;

	_frameCount++;
	_frameTime += _deltaTime;

	if (_frameTime > 1.f)
	{
		_fps = static_cast<uint32>(_frameCount / _frameTime);

		_frameTime = 0.f;
		_frameCount = 0;
	}
```

---

## Engine 클래스

- 멤버 변수 및 함수 추가

```cpp
shared_ptr<Input> _input = make_shared<Input>();
shared_ptr<Timer> _timer = make_shared<Timer>();

shared_ptr<Input> GetInput() { return _input; }
shared_ptr<Timer> GetTimer() { return _timer; }
```

&nbsp;

- `Engine::Update()`

```cpp
_input->Update();
_timer->Update();
```

&nbsp;

- `Engine::ShowFps()`
    - 화면에 FPS를 출력한다.

```cpp
uint32 fps = _timer->GetFps();

WCHAR text[100] = L"";
wsprintf(text, L"FPS : %d", fps);

SetWindowText(_window.hwnd, text);
```

&nbsp;

- `EnginePch.h`에 전역 사용을 위한 매크로 추가

```cpp
#define INPUT GEngine->GetInput()
#define DELTATIME GEngine->GetTimer()->GetDeltaTime()
```

---

## Game 클래스

키보드 입력으로 오브젝트를 이동시켜보자.

&nbsp;

- `Game::update()`
    - 키보드 입력에 따라 `속도 * 시간` 값으로 오브젝트가 움직이도록 한다.

```cpp
static Transform t{};

if (INPUT->GetButton(KEY_TYPE::W))
	t.offset.y += 1.f * DELTATIME;
if (INPUT->GetButton(KEY_TYPE::S))
	t.offset.y -= 1.f * DELTATIME;
if (INPUT->GetButton(KEY_TYPE::A))
	t.offset.x -= 1.f * DELTATIME;
if (INPUT->GetButton(KEY_TYPE::D))
	t.offset.x += 1.f * DELTATIME;

mesh->SetTransform(t);
mesh->Render();
```

&nbsp;

- 결과
    - 키보드 입력에 따라 흰둥이가 잘 움직인다.

![InputAndTimerResult](/assets/images/post_img/directx/InputAndTimerResult.gif)

---

[Input과 Timer 커밋](https://github.com/chaeeun-dev/DirectX12/commit/fa930efe5525ed65f077d71ad4753b2feaf7f6db)

---

## 마치며

이번 시간에는 Input과 Timer 클래스를 구현해보았다. 드디어 게임 세계 속 객체들을 움직일 수 있게 됐다. 앞으로도 파이팅 ㅎㅎ

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*