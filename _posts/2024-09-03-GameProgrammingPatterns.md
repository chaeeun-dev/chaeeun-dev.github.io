---
title: "[GPP] 게임 프로그래밍 패턴"
date: 2024-09-03 12:50:00 +09:00 
categories: BOOK
author_profile: false
---

## Part1 도입

### chapter1 - 구조, 성능, 게임(24.09.03)

**좋은 소프트웨어 구조란?** - ‘구조는 변경과 관련이 있다.’ 좋은 SW 구조는 쉽게 변경(수정) 할 수 있어야 한다.

**코드를 고치는 방법** - 문제가 무엇인지, 어디서 발생하는지 이해한 후 고치고 깔끔하게 통합하기

**디커플링은 어떻게 도움이 되는가?** - (cf. 커플링 : 양쪽 코드 중 한쪽이 없으면 코드를 이해할 수 없는 것) 디커플링의 핵심 목표는 작업에 들어가기 전 알아야할 지식의 양을 줄이는 것, 어느 한 코드를 변경했을 때 다른 코드를 변경하지 않도록 하는 것

**비용은?** - 좋은 구조는 생산성을 크게 높여주지만, 좋은 구조를 만들고 유지하기 위해 많은 노력과 원칙이 필요하다.

**성능과 속도** - 프로그램이 유연하면 성능상 비용이 발생하고, 코드를 최적화하면 유연성이 떨어진다.

**나쁜 코드의 장점** - 기획 확인 용도로 사용할 코드에 사용하면 좋지만, 나중에 확실히 버릴 수 있어야 한다.

**단순함** - 초보 개발자는 모든 유스케이스를 처리하기 위해 생각나는 대로 조건을 마구 늘여놓는다. 예상과 조금만 다른 입력이 들어와도 전혀 작동하지 않는다. 따라서 적은 로직만으로 많은 유스케이스를 정확하게 처리할 수 있어야 한다(규칙 찾는 게 어렵지만, 찾는다면 엄청난 성취감).

**조언** 

- 추상화와 디커플링을 잘 활용하면 코드를 점차 쉽고 빠르게 만들 수 있다. 하지만, 지금 고민 중인 코드에 유연함이 필요하다는 확신이 없다면 추상화와 디커플링을 적용하느라고 시간 낭비하지 말자.
- 개발 내내 성능을 고민하고, 최적화에 맞게 설계해야 한다. 하지만 가정을 코드에 박아 넣어야 하는 저수준의 핵심 최적화는 가능하면 늦게 하라.
- 게임 기획 내용을 확인해볼 수 있도록 빠르게 개발하되, 너무 서두르느라 코드를 엉망으로 만들지 말자. 결국 그 코드도 작업해야 하는 건 우리다.
- 나중에 버릴 코드를 잘 만들겠다고 시간 낭비하지 말자. 록 스타들이 호텔 방을 어지르는 이유는 다음 날 계산하고 나가면 그만이라는 것을 알기 때문이다.
- 무엇보다, 뭔가 재미있는 걸 만들고 싶다면 먼저 만드는 데에서 재미를 느껴보라.


## Part2 디자인 패턴 다시 보기

### ch2 명령(24.09.06)

**명령(command) 패턴은 메서드 호출을 실체화(reify)한 것이다.**

(실체화 - 실제하는 것으로 만든다. 프로그래밍에서는 무엇인가를 ‘일급(first-class)로 만든다는 뜻으로도 통한다.)

cf) 함수(function)와 메서드(method)의 차이 - 메서드는 함수 중에서 클래스의 멤버 함수를 뜻하는 것. 객체에 종속적이다.

→ 함수 호출을 객체로 감쌌다(콜백, 함수 포인터 등).

**입력키 변경**

```cpp
void InputHandler::handleInput() {
        if (isPressed(BUTTON_X)) jump();    // 직접 함수를 호출
        else if (isPressed(BUTTON_Y)) fireGun();
        else if (isPressed(BUTTON_A)) swapWeapon();
        else if (isPressed(BUTTON_B)) loadGun();
}
```

이 코드는 유저 입력을 받아 게임에서 의미 있는 행동으로 전환한다. 그런데 많은 게임이 키를 바꿀 수 있게 해준다. 키 변경을 지원하려면 jump(), fireGun() 함수를 직접 호출하지 말고, 교체 가능한 무엇인가로 바꿔야 한다.

```cpp
class Command
{
public:
	virtual ~Command() { };
	virtual void execute() = 0;
};
```

행동을 실행할 수 있는 공통 상위(부모) 클래스를 정의한다.

```cpp
class JumpCommand : public Command
{
public: 
	virtual void execute() { jump(); }
};

class FireCommand : public Command
{
public:
	virtual void execute() { FireGun(); }
};
```

행동 별로 하위(자식) 클래스를 만든다. 

```cpp
class InputHandler
{
public:
	void handleInput();
	// 명령을 바인드 할 메서드들...
	
private:
	Command* buttonX_;
	Command* buttonY_;
	Command* buttonA_;
	Command* buttonB_;
};
```

입력 핸들러 코드에 각 버튼별로 Command 클래스 포인터를 저장한다. 

```cpp
void InputHandler::handleInput()
{
	if (isPressed(Button_X)) button_X->execute();    // 직접 함수를 호출하지 않고 한 겹 우회
	else if (isPressed(Button_Y)) button_Y->execute();
	else if (isPressed(Button_A)) button_A->execute();
	else if (isPressed(Button_B)) button_B->execute();
}
```

이렇게 handleInput() 함수를 정의해준다. 직접 함수를 호출하지 않고 한 겹 우회하는 계층이 생겼다. 

**액터에게 지시하기**

위의 예제는 잘 동작하지만, jump(), fireGun() 같은 전역 함수가 플레이어 객체를 찾아야 한다는 점에서 커플링이 깔려 있다고 할 수 있다(커플링으로 인해 Command 클래스의 유용성이 떨어짐). 또, JumpCommand 클래스는 오직 플레이어 캐릭터만 점프하게 만들 수 있다. 유연하게 하기 위해 객체를 직접 찾게 하지 말고 밖에서 전달해주자.

```cpp
class Command
{
public:
	virtual ~Command() {}
	virtual void execute(GameActor& actor) = 0;   // GameActor는 게임 객체 클래스
};

class JumpCommand : public Command
{
public:
	virtual void execute(GameActor& actor) { actor.jump(); }
}
```

이렇게 하면 JumpCommand 클래스 하나로 게임에 등장하는 모든 객체에서 jump() 할 수 있다. 남은 것은 입력 핸들러에서 입력을 받아 메서드를 호출하는 명령 객체를 연결하는 코드이다. 

```cpp
Command* InputHandler::handlerInput()
{
	if (isPressed(BUtton_X)) return button_X;
	if (isPressed(BUtton_Y)) return button_Y;
	if (isPressed(BUtton_A)) return button_A;
	if (isPressed(BUtton_B)) return button_B;
	
	return NULL;
}

```