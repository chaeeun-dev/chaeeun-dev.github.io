---
title: "[GPP] 게임 프로그래밍 패턴"
date: 2024-09-03 12:50:00 +09:00 
categories: BOOK
author_profile: false
toc : true
---



# Part1 도입


## chapter1 - 구조, 성능, 게임(24.09.03)

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




# Part2 디자인 패턴 다시 보기


## ch2 명령(24.09.06)

**명령(command) 패턴은 메서드 호출을 실체화(reify)한 것이다. → 함수 호출을 객체로 감쌌다(콜백, 함수 포인터 등).**

(실체화 - 실제하는 것으로 만든다. 프로그래밍에서는 무엇인가를 ‘일급(first-class)로 만든다는 뜻으로도 통한다.)

cf) 함수(function)와 메서드(method)의 차이 - 메서드는 함수 중에서 클래스의 멤버 함수를 뜻하는 것. 객체에 종속적이다.



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

**액터에게 지시하기** - 위의 예제는 잘 동작하지만, jump(), fireGun() 같은 전역 함수가 플레이어 객체를 찾아야 한다는 점에서 커플링이 깔려 있다고 할 수 있다(커플링으로 인해 Command 클래스의 유용성이 떨어짐). 또, JumpCommand 클래스는 오직 플레이어 캐릭터만 점프하게 만들 수 있다. 유연하게 하기 위해 객체를 직접 찾게 하지 말고 밖에서 전달해주자.

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
Command* InputHandler::handleInput()
{
	if (isPressed(BUtton_X)) return button_X;
	if (isPressed(BUtton_Y)) return button_Y;
	if (isPressed(BUtton_A)) return button_A;
	if (isPressed(BUtton_B)) return button_B;
	
	return NULL;		// 아무것도 누르지 않았다면, 아무것도 하지 않는다.
}

```

어떤 액터를 매개변수로 넘겨줘야 할지 모르기 때문에 handleInput()에서는 명령을 실행할 수 없고 함수 호출 시점을 지연한다.


```cpp
Command* command = InputHandler.handleInput();
if (command)
{	
	command->execute(actor);
}
```

이렇게 코드를 작성하면, 명령을 실행할 때 액터만 바꾸면 어떤 액터라도 제어할 수 있게 된다.

플레이어가 제어하는 캐릭터 외에 게임에는 다른 캐릭터가 많다. 이런 캐릭터는 AI가 제어하는데, 같은 명령 패턴을 AI 엔진과 액터 사이에 인터페이스 용으로 사용할 수 있다. Command 객체를 선택하는 AI와 이를 실행하는 액터를 디커플링함으로써 코드가 훨씬 유연해질 수 있다.



**실행취소와 재실행** - 그냥 실행 취소 기능을 구현하려면 굉장히 어렵지만, 명령 패턴을 이용하면 쉽게 만들 수 있다. 어떤 유닛을 옮기는 명령에 대해 알아보자.

```cpp
class MoveUnitCommand : public Command
{
public:
	MoveUnitCommand(Unit* unit, int x, int y) : unit_(unit), x_(x), y_(y) {}

	virtual void execute() { unit_->moveTo(x_, y_); }

private:
	Unit* unit_;
	int x_;
	int y_;
};
```

이전 예제에서는 명령에서 변경하려는 액터와 명령 사이를 추상화로 격리시켰지만, 이 코드는 이동하려는 유닛과 위치 값을 생성자에서 받아 명령과 명시적으로 바인드했다. 


실행 취소 코드를 구현해보자.
```cpp
Class Command
{
public:
	virtual ~Command() {}
	virtual void execute() = 0;
	virtual void undo() = 0;	// 명령을 취소할 수 있도록 순수 가상 함수 undo()를 정의한다.
};
```


MoveUnit Command 클래스에 실행 취소 기능을 넣어보자.

```cpp
class MoveUnitCommand : public Command
{
public:
	MoveUnitCommand(Unit* unit, int x, int y) : unit_(unit), x_(x), _y(y), xBefore_(0), yBefore_(0), x_(x), y_(y) { }

	virtual void execute() 
	{ 	
		// 나중에 이동을 취소할 수 있도록 원래 유닛 위치를 저장한다.
		xBefore_ = unit_->x();
		yBefore_ = unit_->y();
		unit_->moveTo(x_, y_);
	}

	virtual void undo() { unit_->moveTo(xBefore_, yBefore_); }

private:
	Unit* unit;
	int x_, y_;
	int xBefore_, yBefore_;
```

실행 취소 스택 - 실행 취소를 선택하면 현재 명령을 취소하고 현재 명령을 가리키는 포인터를 뒤로 이동... 이런 식으로 구현할 수 있음


**클래스만 좋고 함수형은 별로인가?** - (이 부분은 이해가 잘 안 가서 패스) 





## ch3 경량(24.09.08)


**숲에 들어갈 나무들** - 숲을 실시간 게임으로 구현한다는 것은 수천 그루가 넘는 나무를 각각 수천 폴리곤의 형태로 표현해야한다는 것이다. 나무의 정보를 코드로 표현해면 다음과 같다.

```cpp
class Tree
{
private:
	Mesh mesh_;
	Texture bark_;
	Texture leaves_;
	Vector position_;
	double height_;
	double thcikness_;
	Color barkTint_;
	Color leafTint_;
};
```

많은 객체로 이루어진 숲 전체 데이터를 1프레임에 GPU로 모두 전달하기엔 양이 너무 많다. 그런데 핵심은 숲의 나무가 대부분 비슷해 보인다는 점이다. 객체를 반으로 쪼개 이런 점을 명시적으로 모델링할 수 있다. 모든 나무가 다 같이 사용하는 데이터를 뽑아내 새로운 클래스에 모아보자.

```cpp
class TreeModel
{
private:
	Mesh mesh_;
	Texture bark_;
	Texture leaves_;
};
```

TreeModel 객체는 하나만 존재하게 된다(같은 메시와 텍스처를 여러 번 메모리에 올릴 필요가 없어서). 각 나무 인스턴스는 공유 객체인 TreeModel을 참조하기만 한다.

```cpp
class Tree
{
private:
	TreeModel* model_;
	Vector position_;
	double height_;
	double thcikness_;
	Color barkTint_;
	Color leafTint_;
}
```

![image](https://github.com/user-attachments/assets/9d7507d9-74ea-4083-99af-10108e278099)

나무 인스턴스 각각이 모델 하나를 공유한다. 주메모리에 객체를 ‘저장’하기 위한 것이라면 충분하지만, ‘렌더링’은 다른 문제다. 화면에 숲을 그리기 위해서는 먼저 데이터를 GPU로 전달해야 한다. 어떤 식으로 자원을 공유하고 있는지 그래픽 카드가 이해할 수 있는 방식으로 표현해야 한다.

**수천 개의 인스턴스** - GPU로 보내는 데이터 양을 최소화하기 위해서는 공유 데이터인 TreeModel을 딱 ‘한 번’만 보낼 수 있어야 한다. 그 후 나무마다 값이 다른 정보(위치, 색, 크기)를 전달하고, 마지막으로 GPU에 전체 나무 인스턴스를 그릴 때 공유 데이터를 사용하라고 하면 된다. 다행히 Direct3D, OpenGl 모두 ‘인스턴스 렌더링’을 지원한다.

**경량 패턴** - 경량  패턴은 어떤 객체의 수가 너무 많아 좀 더 가볍게 만들고 싶을 때 사용한다. 경량 패턴은 객체 데이터를 두 종류로 나누는데, 자유문맥(고유 상태)(모든 객체 데이터 값이 같아 공유할 수 있는 데이터)와 외부상태(인스턴스 별로 값이 다른 데이터 ex. 위치, 크기, 색)

**지형 정보** - 땅은 보통 풀, 흙, 언덕, 호수, 강 같은 다양한 지형을 이어 붙여서 만든다(여기서는 타일 기반으로 만들 것). 즉 땅은 작은 타일들이 모여 있는 거대한 격자이다. 지형 종류에는 플레이에 영향을 주는 여러 속성이 있다(e.g. 플레이어가 빠르게 이동할 수 있는지 결정하는 이동 비용 값, 보트로 건널 수 있는지 여부, 렌더링 할 때 사용할 텍스처 등)

```cpp
// 지형 종류를 열거형으로 저장하는 게 일반적
enum Terrain
{
	TERRAIN_GRASS,
	TERRAIN_HILL,
	TERRAIN_RIVER,
	// 기타 등등
};

// 월드는 지형을 거대한 격자로 관리
class World
{
private:
	Terrian tiles_[WIDTH][HEIGHT];
};

// 타일 관련 데이터를 얻는 함수
int World::GetMovementCost(intx, int y)
{
	switch (tiles_[x][y])
		{
		case TERRAIN_GRASS : return 1;
		case TERRAIN_HILL : return 2;
		case TERRAIN_RIVER : return 3;
		// 기타 등등
		}
}
```

이렇게 코드를 작성하는 것보다 데이터를 하나로 합쳐서 캡슐화하는 게 좋다. 아래처럼 지형 클래스를 따로 만드는 것이다. 

```cpp
class Terrain
{
public:
	Terrain(int movementCost, bool isWater, Texture texture)
	: movementCost_(movementCost), isWater(isWater), texture_(texture) {}
	
	// const를 사용한 이유? Terrain 객체를 여러 곳에서 공유해서 쓰기 때문에..(바꾸면 다른 객체에 영향)
	// 경량 객체는 변경 불가능한 상태로 만드는 게 전부이다.
	int GetMovementCost() const { return movementCost_; }
	bool isWater() const { return isWater_; }
	const Texure& getTexture() const { return texture_; }
	
private:
	int moveCost_;
	bool isWater_;
	Texture texture_;
};	
```

이 클래스에는 위치와 관련된 내용은 전혀 없다. 즉 ‘자유 문맥’에 해당한다. 따라서 지형 종류별로 Terrain 객체가 여러 개 있을 필요가 없다. World 클래스 격자 멤버 변수에 열거형이나 Terrain 객체 대신 Terrain 객체 포인터를 넣을 수 있다.

```cpp
class World
{
private:
	Terrain* tiles_[WIDTH][HEIGHT];
	// 그 외...
};
```

지형 종류가 같은 타일들은 모두 같은 Terrain 인스턴스 포인터를 갖는다.

![image](https://github.com/user-attachments/assets/1c08b1ab-81c9-46e8-8caa-2f78fb80081a)

Terrain 인스턴스가 여러 곳에서 사용되다 보니, 동적으로 할당하면 생명 주기를 관리하기 어려워 World 클래스에 저장한다.

```cpp
class World
{
public:
	World() :
	grassTerrain_(1, false, GRASS_TEXTURE), hillTerrain(3, false, HILL_TEXTURE), riverTerrain(2, false, RIVER_TEXTURE) {}
	
private:
	Terrain grassTerrian_;
	Terrain hillTerrain_;
	Terrain riverTerrain_;
	// 그 외...
}
```

이렇게 땅 위를 채울 수 있다.

```cpp
void World::generateTerrain()
{
	//땅에 풀 채우기
	for (int x = 0; x < WIDTH; x++)
		{
			for (int y = 0; y < HEIGHT; y++)
			{
				// 언덕을 몇 개 놓는다.
				if (random(10) == 0)
					tiles_[x][y] = &hillTerrain_;
				else
					tiles_[x][y] = &grassTerrain_;
			}
		}
		
		// 강을 하나 놓는다.
		int x = random(WIDTH);
		for (int y = 0; y < HEIGHT; y++)
			tiles_[x][y] = &riverTerrain_;
}
```

d이제 지형 속성 값을 World의 메서드 대신 Terrain 객체에서 바로 얻을 수 있다. World 클래스는 더 이상 지형의 세부 정보와 커플링 되지 않는다. 타일 속성은 Terrain 객체에서 바로 얻을 수 있다. 

**성능에 대하여** - 포인터냐 열거형이냐에 대한 얘기는 뒤로 하고, 확실한 것은 경량 객체를 한 번은 고려해봐야 한다는 점이다. 경량 패턴을 사용하면 객체를 마구 늘리지 않으면서도 객체지향 방식의 장점을 취할 수 있다.


## ch4 관찰자(24.09.09~10)

관찰자 패턴은 GoF 패턴 중에서도 가장 널리 사용되고 잘 알려졌지만, 세상을 등지고 살아가는 게임 개발자에게는 생판 처음 듣는 얘기일 수도 있다. 오랫동안 골방에 틀어박혀 게임만 만들어온 개발자들을 위해 예제부터 살펴보자. (내용이 웃겨서 넣어봄…)

**업적 달성** - 업적(achievement) 시스템에서 업적은 종류가 광범위하고 달성 방법도 다양해서 깔끔하게 구현하기 어렵다. 특정 기능을 담당하는 코드는 항상 한데 모아두는 게 좋다. 문제는 업적을 여러 게임 플레이 요소에서 발생시킬 수 있다는 점인데, 커플링되지 않고 업적 코드가 동작하려면? 관찰자 패턴을 쓰면 된다. 관찰자 패턴을 적용하면 어떤 코드에서 흥미로운 일이 생겼을 때 누가 받든 상관 없이 알림을 보낼 수 있다. 

물리 코드에서 ‘이제 방금 떨어지기 시작했으니 누군지는 몰라도 알아서 하시오’ 이런 식의 코드를 추가한다(물리 코드는 누가 받든 말든 계속 알림을 보낸다). 업적 시스템은 물리 엔진이 알림을 보낼 때마다 받을 수 있도록 스스로를 등록한다. 물리 코드가 아닌 업적 시스템이 캐릭터, 행동 등을 확인해 업적을 잠금 해제 한다. 

**작동 원리** - Observer와 Achievement 코드를 살펴보자.

```cpp
class Obeserver
{
public: 
	virtual ~Observer() {}
	virtual void onNotify(const Entity& entity, Event event) = 0;   // = 0 <- 순수 가상 함수(구현이 없는 가상 함수)
};

class Achievements : public Observer
{
public: 
	virtual void onNotify(const Entity& entity, Event evnt)
	{
		switch (event)
		{
		case EVENT_ENTITY_FELL :
			if (entity.isHero() && heroIsOnBridge_)
				unlock(ACHIEVEMENT_FEEL_OFF_BRIDGE);
				break;
				// 그 외 다른 이벤트를 처리하고...
				// heroIsOnBridge_ 값을 업데이트한다. 
		}
	}

private:
	void unlock(Achievement achievement)
	{
		// 아직 업적이 잠겨 있다면 잠금 해제한다...
	}
	bool heroIsOnBridge_;
};
```

**대상** - 알림 메서드는 관찰당하는 객체가 호출되는데, 이 객체를 ‘대상’이라고 부른다. 대상의 임무 첫 번째 중 하나는 알림을 끈질기게 기다리는 관찰자 목록을 들고 있는 일이다. 

```cpp
class Subject
{
public:
	void addObserver(Observer* observer)
	{
		// 배열에 추가한다...
	}
	
	void removeObserver(Observer* observer)
	{
		// 배열에서 제거한다...
	}
	
private:
	Observer* observers_[MAX_OBSERVERS];
	int numObservers_;
};
```

중요한 것은 관찰자 목록을 밖에서 변경할 수 있도록 public으로 열어놨다는 점이다. 이를 통해 누가 알림을 받을 것인지를 제어할 수 있다. 대상은 관찰자와 상호작용하지만, 디커플링 되어 있다(물리 코드 어디에도 업적 관련 부분은 없지만 업적 시스템에 알림을 보낼 수 있다). 이 것이 바로 관찰자 패턴의 장점이다. 

대상이 관찰자를 여러 개의 목록으로 관리한다는 점도 중요한데, 자연스럽게 관찰자들은 서로 커플링되지 않게 된다(관찰자를 하나만 지원한다면 두 개 이상의 시스템이 서로를 방해할 것 - 하나를 사용하면 다른 하나를 사용하지 못하는 방식으로). 관찰자를 여러 개 등록할 수 있게 하면 관찰자들이 각자 독립적으로 다뤄지는 걸 보장할 수 있다. 관찰자는 월드에서 같은 대상을 관찰하는 다른 관찰자가 있는지 모른다. 대상의 다른 임무는 알림을 보내는 것이다. 

**물리 관찰** - 남은 작업은 물리 엔진에 훅(hook)을 걸어 알림을 보낼 수 있게 하는 일과 업적 시스템에서 알림을 받을 수 있도록 스스로를 등록하게 하는 일이다. 

```cpp
class Physics : public Subjct
{
public:
	void updateEntity(Entity& entity);
};
```

Subject를 상속받은 Physics 클래스는 notify()를 통해서 알림을 보낼 수 있지만, 밖에서는 notify()에 접근할 수 없다. 이제 물리 엔진에서 중요한 일이 생기면 notify()를 호출해 전체 관찰자에게 알림을 전달하여 일을 처리하게 한다. 

**관찰자 패턴의 불평거리? 너무 느려** - 정적 호출보다 약간 느리긴 하지만, 성능에 민감한 코드가 아니라면 문제가 되지 않는다. 

**너무 빠르다고?** - 주의해야 할 점은 관찰자 패턴이 동기적이라는 점이다. 대상이 관찰자 메서드를 직접 호출하기 때문에 모든 관찰자가 알림 메서드를 반환하기 전에는 다음 작업을 진행할 수 없다(실제로 큰 일은 아니니 알고만 있으면 된다). 관찰자를 멀티 스레드, 락과 함께 사용할 때는 조심해야 한다. 어떤 관찰자가 대상의 락을 물고 있다면 게임 전체가 교착 상태에 빠질 수 있다. 

**동적 할당을 너무 많이 해** - 관찰자가 추가될 때만 메모리를 할당하는데, 알림을 보낼 때는 메서드 호출만 일어나지 동적 할당은 전혀 하지 않는다.

**관찰자 연결 리스트** - 지금까지의 코드에서 Observer 클래스 자신은 이들 포인터 목록을 참조하지 않는다. 하지만 Observer에 상태를 조금 추가하면 관찰자가 스스로를 엮게 만들어 동적 할당 문제를 해결할 수 있다. 대상에 포인터 컬렉션을 따로 두지 않고, 관찰자 객체가 연결 리스트의 노드가 되는 것이다(대상(HEAD_) → 관찰자(NEXT_) → 관찰자(NEXT_)). 아래는 동적 할당 없이 관찰자 연결리스트로 추가, 삭제, 알림 보내기를 수행하는 코드이다.

```cpp
class Subject    // Subject 클래스에 관찰자 연결 리스트의 첫 째 노드를 가리키는 포인터 추가
{
	Subject() : head(NULL) {}
	
private:
	Observer* head_;
}

class Observer   // Observer에 연결 리스트의 다음 관찰자를 가리키는 포인터 추가
{
	friend class Subject;    // friend - Subject가 관리해야 할 관찰자 목록에 접근하기 위한 가장 간단한 방법
	
public:
	Observer() : next_(NULL) {}
	
private:
	Observer* next_;
}

// 단순 연결 리스트
// 노드 추가
void Subject::addObserver(Observer* observer)
{
	// 연결리스트 - 앞 쪽에 노드 추가하는 방식(A-B-C 순으로 추가했으면 C-B-A 순으로 알림 -> 순서로 인한 의존 관계가 없게 만들어야 함!)
	observer->next = head;
	head_ = obesever;
}

// 노드 제거 - 단순 연결리스트라서 연결 리스트를 순회해야 함.
void Subject::removeObserver(Observer* observer)
{
	if (head_ == observer)
	{
		head_ = observer->next_;
		observer->next = NULL;
		return;
	}
	
	Observe* current = head_;
	while (current != NULL)
	{
		if (current->next_ == observer)
		{
			current->next_ = observer->next_;
			observer->next_ = NULL;
			return;
		}
		current = current->next_;
	}
}

// 알림 보내기
void Subject::notify(const Entity& entity, Event event)
{
	Observer* ovserver = head_;
	while (observer != NULL)
	{
		observer->onNotify(entity, event);
		observer = observer->next_;
	}
}

```

**리스트 노드 풀** - 전과 마찬가지로 대상이 관찰자 연결 리스트를 들고 있다. 이 연결 리스트의 노드는 관찰자 객체가 아니다. 따로 간단한 노드를 만들어 관찰자와 다음 노드를 포인터로 가리킨다.

![image](https://github.com/user-attachments/assets/03b800d9-3af6-4c45-a925-21dd959b3ae8)

같은 관찰자를 여러 노드에서 가리킬 수 있다는 건 여러 대상을 한 번에 관찰할 수 있다는 것이다. 동적 할당을 피하는 법은 간단한데, 모든 노드가 같은 자료형, 크기이므로 풀(19장)에 미리 할당하면 된다. 고정된 크기이므로 미리 공간을 확보할 수 있어 동적 할당 없이 재사용 할 수 있다. 

**남은 문제점들** - 관찰자 패턴은 간단하고 빠르며 메모리 측면에서도 깔끔하게 만들 수 있는데, 그렇다고 항상 관찰자 패턴을 써야하는 건 아니다. 상황에 맞게 디자인 패턴을 활용해야지, 만능이라고 믿고 남발하면 안 된다. 

**대상과 관찰자 제거** - 예제 코드에서 관찰자를 부주의하게 삭제하면 대상에 있는 포인터가 이미 삭제된 객체를 가리킬 수 있다. 해제된 메모리를 가리키는 dangling pointer에 알림을 보내는 건 문제가 크다. 

대상이 삭제되면 더 이상 알림을 받을 수 없는데도 관찰자는 모르고 알림을 기다릴 수도 있다. 이걸 막기 위해서는 대상이 죽었을 때 대상이 삭제되기 직전 마지막으로 ‘사망’알림을 보내면 된다. 사망 알림을 받은 관찰자는 알아서 필요한 작업을 수행하면 된다. 

관찰자는 제거하기 어려운데, 대상이 관찰자를 포인터로 알고 있기 때문이다. 가장 쉬운 방법은 관찰자가 삭제될 때 스스로를 등록 취소하는 것이다. 관찰자는 관찰 대상을 알고 있으므로 소멸자에서 대상의 removeObserver()를 호출하면 된다. 

**GC(**Garbage Collector**)가 있는데 무슨 걱정이람** - 캐릭터가 공격 받으면 UI 창에서 체력바를 갱신한다. 여기서 상태 창을 닫을 때 관찰자를 등록 취소하지 않는다면 어떻게 될까? UI는 안 보이지만 캐릭터의 관찰자 목록에서 여전히 UI를 참조하고 있어 GC가 수거하지 않는다. 상태창을 열 때마다 상태창 인스턴스를 새로 맏들어 관찰자 목록에 추가하기 때문에 관찰자 목록은 점점 커지게 된다. 캐릭터는 계속 모든 상태창 객체에 알림을 보내는데, 화면에 있지도 않은 상태창에 알림을 보내기 때문에 CPU 클럭을 낭비하게 된다(상태창 효과음이 연속해서 날 수도..). 이는 매우 빈번한 문제인데, ‘사라진 리스너 문제’라고 부른다. 등록 취소는 주의하자!

**무슨 일이 벌어진 거야?** - 관찰자 패턴을 사용하는 이유는 두 코드 간의 결합을 최소화하기 위해서다. 이 덕분에 대상은 다른 관찰자와 정적으로 묶이지 않고도 간접적인 상호작용을 할 수 있다. (뒤에는 이해가 안 가서 읽기만 하고 다음으로 패스..)

**오늘날의 관찰자** - 클래스를 계속 상속 받는 건 무겁고 융통성이 없다. 최신 방식은 메서드나 함수 레퍼런스만으로 ‘관찰자’를 만드는 것이다. 필자가 관찰자 패턴을 다시 만든다면 클래스보다 함수형 방식으로 만들 거라고 한다. 

**미래의 관찰자** - 관찰자 패턴 관련 코드의 공통점 : 1. 어떤 상태가 변했다는 알림을 받는다. 2. 이를 반영하기 위해 UI 상태 일부를 바꾼다 → 체력이 7이면 체력바 너비를 70픽셀로 바꾸는 식이다. 이런 방법 보다 ‘데이터 바인딩’이 인기를 얻고 있는데 이는 어떤 값이 변경되면 관련된 UI 요소나 속성을 바꿔줘야 하는 귀찮은 작업을 알아서 해준다. UI는 성능에 덜 민감하기에 데이터 바인딩이 느리더라도 대세가 될 것이라고 한다.



## ch5. 프로토타입(24.09.11)

**프로토타입 디자인 패턴** - 몬스터는 종류마다 스포너가 따로 있다. (cf.스포너 spawner) 

```cpp
class Monster
{
	// ect...
};

class Ghost : public Monster {};
class Demon : public Monster {};
class Sorcerer : public Monster {};
```

한 가지 스포너는 한 가지 몬스터 인스턴스만 만든다. 몬스터 클래스마다 스포너 클래스를 만들면 스포너 클래스 상속 구조가 몬스터 클래스 상속 구조를 따라가게 된다.

![image](https://github.com/user-attachments/assets/bbeb071d-623c-4761-b133-ea0f29e0abca)

```cpp
class Spawner 
{
public:
	virtual ~Spawner() {}
	virtual Monster* spawnMonster() = 0;
};

class GhostSpawner : public Spawner
{
public:
	virtual Monster* spawnMonster()
	{
		return new Ghost();
	}
};

class DemonSpawner : public Spawner
{
public:
	virtual Monster* spawnMonster()
	{
		return new Demon();
	}
};
```

이 코드는 별로다. 클래스도 많고, 행사 코드(프로그램의 실행과 직접적으로는 관계가 없는 프로그래밍 문법적 서식)도 많고, 반복 코드도 많다. 이걸 프로토타입 패턴으로 해결할 수 있다. 핵심은 어떤 객체가 자기와 비슷한 객체를 스폰할 수 있다는 점이다. 유령 객체 하나로 다른 유령 객체를 여럿 만들 수 있는데, 어떤 몬스터 객체든 자신과 비슷한 몬스터 객체를 만드는 원형(protptypal) 객체로 사용할 수 있다. 

```cpp
class Monster
{
public:
	virtual ~Monster() {]
	
	// Monster 하위 클래스는 자신과 자료형과 상태가 같은 새로운 객체를 반환하도록 clone()을 구현한다.
	virtual Monster* clone() = 0;  
	}

class Ghost : public Monster
{
public:
	Ghost(int health, int speed) : health_(health), speed_(speed) {}
	
	virtual Moster* clone() { return new Ghost(health, speed); }
	
private:
	int health_;
	int speed_;
}

// Monster를 상속받는 모든 클래스에 clone() 메서드가 있다면 스포너 클래스를 종류별로 만들 필요 없이 하나만 만들면 된다. 
class Spawner
{
public:
	Spawner(Monster* prototype) : prototype_(prototype) {}
	Monster* spawnerMonster() { return prototype_->clone(); }
}
```

Spawner 클래스 내부에는 Monster 객체가 숨어 있다. 이 객체는 자기와 같은 Monster 객체를 도장 찍듯 만들어내는 스포너 역할만 한다. 

![image](https://github.com/user-attachments/assets/3f6315d1-8324-4e29-81eb-702865b92abd)

유령 스포너를 만들려면 원형으로 사용할 유령 인스턴스를 만든 후에 스포너에 전달한다. 

```cpp
Monster* ghostPrototype = new Gohst(15, 3);
Spawner* ghostSpawner = new Spawner(ghostPrototype);
```

프로토타입 패턴의 좋은 점은 프로토타입의 클래스 뿐만 아니라 상태도 같이 복제(clone)한다는 점이다. 

**얼마나 잘 작동하는가?** - 이제 몬스터마다 스포너 클래스를 따로 만들지 않아도 된다. 그래도 Monster 클래스마다 clone()을 구현해야 하기 때문에 코드 양은 별 차이가 없다. clone()을 만들다보면 객체를 깊은 복사 해야 할지, 얕은 복사를 해야할지, 악마가 삼지창을 들고 있다면 그것도 복제해야 할지 애매한 경우가 있다. 프로토타입 패턴을 써도 코드 양이 많이 줄지 않고, 요즘 게임 엔진들은 몬스터마다 클래스를 따로 만들지 않는다. 오랜 삽질을 통해 클래스 상속 구조가 복잡하면 유지보수가 힘들다는 걸 체득했기 때문에, 요즘은 개체 종류별로 클래스를 만들기보다 컴포넌트나 타입객체로 모델링 하는 것을 선호한다. 

**스폰함수** - 스폰 함수를 만들어보자.

```cpp
Monster* spawnGhost() { return new Ghost; }
```

몬스터 종류마다 클래스를 만드는 것보다 행사코드가 훨씬 적다. 이제 스포너 클래스에는 함수 포인터 하나만 두면 된다.

```cpp
typedef Monster* (*SpawnCallback)();

class Spawner
{
public:
	Spawner(SpawnCallback spawn) : spawn_(spawn) {}
	Monster* spawnMonster() { return spawn_(); }

private:
	SpawnCallback spawn_;
};

// 유령을 스폰하는 객체는 이렇게 만들 수 있다.
Spawner* ghostSpawner = new Spawner(spawnGhost);
```


**템플릿** - 스포너 클래스를 이용해 인스턴스를 생성하고 싶지만 특정 몬스터 클래스를 하드코딩하기 싫다면 몬스터 클래스를 템플릿 타입 매개 변수로 전달하면 된다.

```cpp
class Spawner
{
public:
	virtual ~Spawner() {}
	virtual Monster* spawnMonster() = 0;
};

template <class T>
class SpawnerFor : public Spawner
{
public:
	virtual Monster* spawnMonster() { return new T(); } 
}

// 사용법
Spawner* ghostSpawner = new SpawnerFor<Ghost>();
```

**일급 자료형** - 스폰 함수와 템플릿 이 두 가지의 방법을 통해 Spawner 클래스에 자료형을 매개변수로 전달할 수 있다. C++에서는 자료형이 일급이 아니다 보니 그런 것. 자바스크립트, 파이썬, 루비 등 클래스가 전달 가능한 일급 자료형인 동적 자료형 언어에서는 이 문제를 훨씬 직접적으로 풀 수 있다. 

**프로토타입 언어 패러다임** - oop는 곧 클래스라고 하는 사람이 많지만… oop의 가장 큰 특징은 상태와 동작을 함께 묶는 데 있다. 클래스 이외에 셀프가 있는데, 셀프에서는 oop에서 할 수 있는 걸 다 할 수 있지만 클래스는 없다.

**셀프** - 의미만 본다면 셀프는 클래스 기반 언어보다 더 객체지향적이다. oop를 상태와 동작을 같이 묶어놓은 것이라고 할 때, 클래스 기반 언어는 상태와 동작 사이에 분명한 구별이 있다. 상태는 클래스가 아닌 인스턴스에 들어 있다. 반대로 메서드를 호출할 때는 인스턴스의 클래스를 찾는다. 즉 동작은 클래스에 있다. 항상 한 단계를 거쳐서 메서드를 호출한다는 점에서 필드(상태)와 메서드는 다르다. 셀프는 이런 구별이 없고, 무엇이든 객체에서 바로 찾을 수 있다. 

클래스 기반의 언어에서 상속은 단점도 있지만, 다형성을 통해 코드를 재사용하고 중복을 줄일 수 있다. 셀프는 클래스 없이 이런 일을 수행하기 위해 ‘위임’ 개념이 있다. 

먼저 해당 객체에서 필드나 메서드를 찾아보고, 있다면 그걸 쓰고 없다면 상위 객체를 찾아 본다(상위 객체는 그냥 다른 객체 레퍼런스일 뿐이다). 첫 번째 객체에 속성이 없다면 상위 객체를 살펴보고 그래도 없다면 객체의 상위 객체에서 찾아보고, 이를 반복한다. 즉 찾아보고 없으면 상위 객체에 위임하는 것이다. 

상위 객체를 통해 동작을 여러 객체가 재사용 할 수 있기 때문에 클래스가 제공하는 기능 대부분을 대신할 수 있다. 클래스의 또 다른 역할은 인스턴스 생성기이다(e.g. new A()). 클래스가 없을 때 내용이 같은 객체를 만드려면? 셀프에서는 복제하면 된다. 셀프는 모든 객체가 프로토타입 디자인 패턴을 저절로 지원하는 것과 다를 게 없다.
