---
title: "[GPP] 게임 프로그래밍 패턴 Part Ⅱ 디자인 패턴 다시 보기"
date: 2024-10-02 12:50:00 +09:00 
categories: GPP
author_profile: false
toc : true
toc_sticky: true
---

# chapter2 - 명령(24.09.06)

**명령(command) 패턴은 메서드 호출을 실체화(reify)한 것이다. → 함수 호출을 객체로 감쌌다(콜백, 함수 포인터 등).**

(실체화 - 실제하는 것으로 만든다. 프로그래밍에서는 무엇인가를 ‘일급(first-class)로 만든다는 뜻으로도 통한다.)

cf) 함수(function)와 메서드(method)의 차이 - 메서드는 함수 중에서 클래스의 멤버 함수를 뜻하는 것. 객체에 종속적이다.



## 2.1 입력키 변경

모든 게임에서는 유저의 입력을 받아 게임에서 의미 있는 행동으로 전환한다.

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

## 2.2 액터에게 지시하기

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


## 2.3 실행취소와 재실행

그냥 실행 취소 기능을 구현하려면 굉장히 어렵지만, 명령 패턴을 이용하면 쉽게 만들 수 있다. 어떤 유닛을 옮기는 명령에 대해 알아보자.

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


## 2.4 클래스만 좋고, 함수형은 별로인가?

(이 부분은 이해가 잘 안 가서 패스) 



# ch3 경량(24.09.08)

## 3.1 숲에 들어갈 나무들

숲을 실시간 게임으로 구현한다는 것은 수천 그루가 넘는 나무를 각각 수천 폴리곤의 형태로 표현해야한다는 것이다. 나무의 정보를 코드로 표현해면 다음과 같다.

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

## 3.2 수천 개의 인스턴스

GPU로 보내는 데이터 양을 최소화하기 위해서는 공유 데이터인 TreeModel을 딱 ‘한 번’만 보낼 수 있어야 한다. 그 후 나무마다 값이 다른 정보(위치, 색, 크기)를 전달하고, 마지막으로 GPU에 전체 나무 인스턴스를 그릴 때 공유 데이터를 사용하라고 하면 된다. 다행히 Direct3D, OpenGl 모두 ‘인스턴스 렌더링’을 지원한다.

## 3.3 경량 패턴

 경량  패턴은 어떤 객체의 수가 너무 많아 좀 더 가볍게 만들고 싶을 때 사용한다. 경량 패턴은 객체 데이터를 두 종류로 나누는데, 자유문맥(고유 상태)(모든 객체 데이터 값이 같아 공유할 수 있는 데이터)와 외부상태(인스턴스 별로 값이 다른 데이터 ex. 위치, 크기, 색)

## 3.4 지형 정보

 땅은 보통 풀, 흙, 언덕, 호수, 강 같은 다양한 지형을 이어 붙여서 만든다(여기서는 타일 기반으로 만들 것). 즉 땅은 작은 타일들이 모여 있는 거대한 격자이다. 지형 종류에는 플레이에 영향을 주는 여러 속성이 있다(e.g. 플레이어가 빠르게 이동할 수 있는지 결정하는 이동 비용 값, 보트로 건널 수 있는지 여부, 렌더링 할 때 사용할 텍스처 등)

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

## 3.5 성능에 대해서

포인터냐 열거형이냐에 대한 얘기는 뒤로 하고, 확실한 것은 경량 객체를 한 번은 고려해봐야 한다는 점이다. 경량 패턴을 사용하면 객체를 마구 늘리지 않으면서도 객체지향 방식의 장점을 취할 수 있다.



# ch4 관찰자(24.09.09~10)

관찰자 패턴은 GoF 패턴 중에서도 가장 널리 사용되고 잘 알려졌지만, 세상을 등지고 살아가는 게임 개발자에게는 생판 처음 듣는 얘기일 수도 있다. 오랫동안 골방에 틀어박혀 게임만 만들어온 개발자들을 위해 예제부터 살펴보자. (내용이 웃겨서 넣어봄…)

## 4.1 업적 달성

업적(achievement) 시스템에서 업적은 종류가 광범위하고 달성 방법도 다양해서 깔끔하게 구현하기 어렵다. 특정 기능을 담당하는 코드는 항상 한데 모아두는 게 좋다. 문제는 업적을 여러 게임 플레이 요소에서 발생시킬 수 있다는 점인데, 커플링되지 않고 업적 코드가 동작하려면? 관찰자 패턴을 쓰면 된다. 관찰자 패턴을 적용하면 어떤 코드에서 흥미로운 일이 생겼을 때 누가 받든 상관 없이 알림을 보낼 수 있다. 

물리 코드에서 ‘이제 방금 떨어지기 시작했으니 누군지는 몰라도 알아서 하시오’ 이런 식의 코드를 추가한다(물리 코드는 누가 받든 말든 계속 알림을 보낸다). 업적 시스템은 물리 엔진이 알림을 보낼 때마다 받을 수 있도록 스스로를 등록한다. 물리 코드가 아닌 업적 시스템이 캐릭터, 행동 등을 확인해 업적을 잠금 해제 한다. 

## 4.2 작동 원리

 Observer와 Achievement 코드를 살펴보자.

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

## 4.3 너무 느려

정적 호출보다 약간 느리긴 하지만, 성능에 민감한 코드가 아니라면 문제가 되지 않는다. 

**너무 빠르다고?** - 주의해야 할 점은 관찰자 패턴이 동기적이라는 점이다. 대상이 관찰자 메서드를 직접 호출하기 때문에 모든 관찰자가 알림 메서드를 반환하기 전에는 다음 작업을 진행할 수 없다(실제로 큰 일은 아니니 알고만 있으면 된다). 관찰자를 멀티 스레드, 락과 함께 사용할 때는 조심해야 한다. 어떤 관찰자가 대상의 락을 물고 있다면 게임 전체가 교착 상태에 빠질 수 있다. 

## 4.4 동적 할당을 너무 많이 해

관찰자가 추가될 때만 메모리를 할당하는데, 알림을 보낼 때는 메서드 호출만 일어나지 동적 할당은 전혀 하지 않는다.

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

## 4.5 남은 문제점들

 관찰자 패턴은 간단하고 빠르며 메모리 측면에서도 깔끔하게 만들 수 있는데, 그렇다고 항상 관찰자 패턴을 써야하는 건 아니다. 상황에 맞게 디자인 패턴을 활용해야지, 만능이라고 믿고 남발하면 안 된다. 

**대상과 관찰자 제거** - 예제 코드에서 관찰자를 부주의하게 삭제하면 대상에 있는 포인터가 이미 삭제된 객체를 가리킬 수 있다. 해제된 메모리를 가리키는 dangling pointer에 알림을 보내는 건 문제가 크다. 

대상이 삭제되면 더 이상 알림을 받을 수 없는데도 관찰자는 모르고 알림을 기다릴 수도 있다. 이걸 막기 위해서는 대상이 죽었을 때 대상이 삭제되기 직전 마지막으로 ‘사망’알림을 보내면 된다. 사망 알림을 받은 관찰자는 알아서 필요한 작업을 수행하면 된다. 

관찰자는 제거하기 어려운데, 대상이 관찰자를 포인터로 알고 있기 때문이다. 가장 쉬운 방법은 관찰자가 삭제될 때 스스로를 등록 취소하는 것이다. 관찰자는 관찰 대상을 알고 있으므로 소멸자에서 대상의 removeObserver()를 호출하면 된다. 

**GC(**Garbage Collector**)가 있는데 무슨 걱정이람** - 캐릭터가 공격 받으면 UI 창에서 체력바를 갱신한다. 여기서 상태 창을 닫을 때 관찰자를 등록 취소하지 않는다면 어떻게 될까? UI는 안 보이지만 캐릭터의 관찰자 목록에서 여전히 UI를 참조하고 있어 GC가 수거하지 않는다. 상태창을 열 때마다 상태창 인스턴스를 새로 맏들어 관찰자 목록에 추가하기 때문에 관찰자 목록은 점점 커지게 된다. 캐릭터는 계속 모든 상태창 객체에 알림을 보내는데, 화면에 있지도 않은 상태창에 알림을 보내기 때문에 CPU 클럭을 낭비하게 된다(상태창 효과음이 연속해서 날 수도..). 이는 매우 빈번한 문제인데, ‘사라진 리스너 문제’라고 부른다. 등록 취소는 주의하자!

**무슨 일이 벌어진 거야?** - 관찰자 패턴을 사용하는 이유는 두 코드 간의 결합을 최소화하기 위해서다. 이 덕분에 대상은 다른 관찰자와 정적으로 묶이지 않고도 간접적인 상호작용을 할 수 있다. (뒤에는 이해가 안 가서 읽기만 하고 다음으로 패스..)

## 4.6 오늘날의 관찰자

 클래스를 계속 상속 받는 건 무겁고 융통성이 없다. 최신 방식은 메서드나 함수 레퍼런스만으로 ‘관찰자’를 만드는 것이다. 필자가 관찰자 패턴을 다시 만든다면 클래스보다 함수형 방식으로 만들 거라고 한다. 

## 4.7 미래의 관찰자

**미래의 관찰자** - 관찰자 패턴 관련 코드의 공통점 : 1. 어떤 상태가 변했다는 알림을 받는다. 2. 이를 반영하기 위해 UI 상태 일부를 바꾼다 → 체력이 7이면 체력바 너비를 70픽셀로 바꾸는 식이다. 이런 방법 보다 ‘데이터 바인딩’이 인기를 얻고 있는데 이는 어떤 값이 변경되면 관련된 UI 요소나 속성을 바꿔줘야 하는 귀찮은 작업을 알아서 해준다. UI는 성능에 덜 민감하기에 데이터 바인딩이 느리더라도 대세가 될 것이라고 한다.



## ch5. 프로토타입(24.09.11~13)

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

상위 객체를 통해 동작을 여러 객체가 재사용 할 수 있기 때문에 클래스가 제공하는 기능 대부분을 대신할 수 있다. 클래스의 또 다른 역할은 인스턴스 생성기이다(e.g. new A()). 클래스가 없을 때 내용이 같은 객체를 만드려면? 셀프에서는 복제하면 된다. 셀프는 모든 객체가 프로토타입 디자인 패턴을 저절로 지원하는 것과 다를 게 없다. 모든 객체가 복제될 수 있기 때문에 비슷한 객체를 여럿 만드려면 이렇게 한다. [ 1) 객체 하나를 원하는 상태로 만들고, 시스템에서 제공하는 기본 Object 객체를 복제한 뒤 필드와 메서드를 채워 넣는다. 2) 원하는 만큼 복제한다.] 셀프에서는 귀찮게 직접 clone() 메서드를 구현하지 않아도 프로토타입 디자인 패턴의 우아함을 시스템적으로 제공한다. 

[Javascript는-프로토타입-기반-언어이다](https://velog.io/@huurray/Javascript%EB%8A%94-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85-%EA%B8%B0%EB%B0%98-%EC%96%B8%EC%96%B4%EC%9D%B4%EB%8B%A4)

(프로토타입 개념에 대한 이해가 잘 안 돼서 구글링 했는데 설명이 잘 돼있어서 이해하는데 도움이 많이 됐다.)

Java, C++ 등 : 클래스 기반 객체지향 프로그래밍 언어

JavaScript : 프로토타입 기반 객체지향 프로그래밍 언어, class 문법이 추가 됐지만 클래스 기반으로 바뀐 것은 아니다. 

프로토타입 개념 - 자바 스크립트에서만 사용되는 게 아니고 하나의 디자인 패턴이다. 특징은 객체를 복제하면서 생성한다는 점이다. 

cf) 필드 : 객체의 정보(상태) / 메서드 : 객체의 동작(기능)

**자바스크립트는 어떤가?** -  자바스크립트를 만든 브렌던 아이크는 셀프로부터 직접 영감을 받았다. 많은 자바스크립트 문법이 프로토타입 기반 방식이다. 객체 속성에는 필드나 메서드가 들어간다. 객체는 프로토타입이라고 부르는 다른 객체를 지정할 수 있어서 자기 자신에게 없는 필드는 포로토타입에 위임할 수 있다. 

저자는 그렇긴 해도 자바스크립트가 프로토타입보다 클래스 기반의 언어라고 생각한다고 한다. 프로토타입 기반 언어의 핵심인 ‘복제’를 찾아볼 수 없기 때문이다. 

정리하자면,

- 자바스크립트에서는 자료형을 정의하는 객체로부터 new를 호출하는 생성자 함수를 통해 객체를 생성한다.
- 상태는 인스턴스 그 자체에 저장된다.
- 동작은 자료형이 같은 객체 모두가 공유하는 메서드 집합을 대표하는 별도 객체인 프로토타입에 저장되고 위임을 통해 간접 접근된다.

프로토타입 방식을 고집하면 코드가 너무 어려워지기 때문에, 자바스크립트가 핵심 문법을 좀 더 클래스스럽게 감싼 게 더 좋아보인다고 말한다.

**데이터 모델링을 위한 프로토타입** - 프로토타입이, 더 정확하게는 위임 개념이 쓸모 있는 분야에 대해 알아보자. 

세월이 지날수록 게임 바이너리에서 코드보다 데이터가 차지하는 용량이 커지고 있다. 요즘 게임에서 코드는 게임을 실행하기 위한 ‘엔진’일 뿐, 게임 관련 콘텐츠는 모두 데이터에 정의되어 있다. 

(…) 프로그래밍 언어를 사용하는 이유는 복잡성을 제어할 수 있는 수단을 가지고 있어서다. 같은 코드를 복붙하는 대신에 하나의 함수로 만들어서 호출한다. 여러 클래스에 같은 메서드를 복사하는 대신 클래스로 만들어 상속 받거나 믹스인(mix in : 상속받지 않고도 다른 클래스에 있는 코드를 재활용하는 기법)한다. 

게임 데이터 규모가 일정 이상이 되면 코드와 비슷한 기능이 필요하다. 데이터 모델링에서 많이 사용하는 기법 중 하나가 JSON이다. 키/값 구조로 이루어진 데이터 객체는 맵, 속성목록 등의 용어로 불리고 있다. 

좋은 프로그래머는 중복을 싫어한다. 하지만 멍청한 JSON에는 그런 기능이 없다. 더 영리하게 만들어보자면 중복되는 데이터를 프로토타입으로 지정해서 반복 입력하지 않아도 되게 만들면 된다.


## ch6 싱글턴(24.09.18~21)

게임 프로그래밍 올인원 강의 듣다가 간단하게 정리했던 static과 singleton 개념 참고 [**[C++] static과 singleton**](https://chaeeun-dev.github.io/c++/static_and_singleton/)

싱글턴은 워낙 남용되는 패턴이다 보니 이 장에서는 싱글턴을 피할 방법을 주로 다룬다.

**싱글턴 패턴** - **오직 한 개의 클래스 인스턴스만 갖도록 보장** : 클래스 인스턴스를 싱글턴으로 만들면 인스턴스를 하나만 가질 수 있게 된다. 외부 시스템과 상호작용하면서 전역 상태를 관리하는 클래스에서 쓰이기도 한다. **전역 접근점을 제공** : 하나의 인스턴스만 생성하는 것에 더해서, 싱글턴 패턴은 그 인스턴스를 전역에서 접근할 수 있는 메서드를 제공한다. 누구든지, 어디서든지 우리가 만든 인스턴스에 접근할 수 있다.

이 모든 걸 제공하는 클래스는 이렇게 구현할 수 있다.

```cpp
class FileSystem
{
public:
	static FileSystem& instance()    // public에 있는 정적 메서드 instance()는 코드 어디에서나 싱글턴 인스턴스에 접근할 수 있게 하고, 싱글턴을 실제로 필요로 할 때까지 인스턴스 초기화를 미루는 역할도 한다. 
	{
		// 게으른 초기화
		if (instance_ == nullptr)
			instance_ = new FileSystem();
		
		return *instance_;
	}
	
private:
	FileSystem() {}
	static FileSystem* instance_;    // instance_ 정적 멤버 변수는 클래스 인스턴스를 저장한다. private이기 때문에 밖에서 생성할 수 없다. 
};
```

요즘에는 이렇게도 만든다

```cpp
class FileSystem
{
public:
	static FileSystem& instance()
	{
		static FileSystem *instance_ = new FileSystem();
		return *instance_;
	}
	
private:
	FileSystem() {}
};

// C++ 11에서는 정적 지역 변수 초기화 코드가 멀티스레드 환경에서도 딱 한 번 실행돼야 한다. 이 코드는 이전 예제외 달리 스레드 안전(thread-safe)하다.
```

**싱글턴을 왜 사용하는가?** - **한 번도 사용하지 않는다면 아예 인스턴스를 생성하지 않는다** : 메모리와 CPU 사용량을 줄일 수 있다. **런타임에 초기화된다** : 보통 싱글턴 대안으로 정적(static) 멤버 변수를 많이 사용한다. 그런데 정적 멤버 변수는 자동 초기화(automatic initialization)되는 문제가 있다. 즉, 컴파일러는 main 함수를 호출하기 전에 정적 변수를 초기화하기 때문에 프로그램이 실행된 다음에야 알 수 있는(파일로 읽어들인 설정 값 같은) 정보를 활용할 수 없다. 정적 변수 초기화 순서도 컴파일러가 보장해주지 않는다. 반면 게으른 초기화(최대한 늦게 초기화)는 이 문제를 해결해준다. 초기화 될 쯤에는 클래스가 필요로 하는 정보가 준비되어 있다. 초기화할 때 다른 싱글턴을 참조해도 괜찮다. **싱글턴을 상속할 수 있다** - 파일 시스템 래퍼가 크로스 플래솜을 지원해야 한다면 추상 인터페이스를 만든 뒤, 플랫폼마다 구체 클래스를 만들면 된다.

[순수 가상 함수 개념 참고 블로그](https://hwan-shell.tistory.com/223)

상위 클래스를 먼저 만든다.

```cpp
class FileSystem    // cf) 추상 클래스(abstract class) - 추상 클래스는 객체로 만들지 못하고 상속으로써만 사용된다. 
{
public:
	virtual ~FileSystem() {}
	virtual char* readFile(char* path) = 0;    // cf) 순수 가상 함수(pure virtual function) - 함수 정의X 함수 선언만 이루어짐
	virtual void writeFile(char* path, char* contents) = 0;
}

// cf) 추상 클래스를 상속받은 자식 클래스는 무조건 해당 순수 가상 함수를 override해야 한다. 
// 무조건 재정의를 해야하는 경우에 사용!
```



플랫폼별로 하위 클래스를 정의한다.

```cpp
class PS3FileSystem : public FileSystem
{
public:
	virtual char* readFile(char* path)
	{
		// 소니의 파일 IO API를 사용한다...
	}
	
	virtual void writeFile(char* path, char* contents)
	{
		// 소니의 파일 IO API를 사용한다...
	}
};

class WIIFileSystem : public FileSystem
{
public:
	virtual char* readFile(char* path)
	{
		// 닌텐도의 파일 IO API를 사용한다...
	}
	
	virtual void writeFile(char* path, char* contents)
	{
		// 닌텐도의 파일 IO API를 사용한다...
	}
};
```

이제 FileSystem 클래스를 싱글턴으로 만든다.

```cpp
class FileSystem
{
public:
	static FileSystem& instance();
	
	virtual ~FileSystem() {};
	virtual char* readFile(char* path) = 0;   
	virtual void writeFile(char* path, char* contents) = 0;

protected:
	FileSystem() {}
};
```

핵심은 인스턴스를 생성하는 부분이다.

```cpp
FileSystem& FileSystem::instance()
{
#if PLATFORM == PLAYSTATION3
	static FIleSystem *instance = new PS3FileSystem();
#elif PLATFORM == WII
	static FIleSystem *instance = new WiiFileSystem();
#endif
	return *instance;
}
// 이렇게 하면 시스템 래퍼에 필요한 기능을 다 제공하는 셈이다.  안정적으로 작동하고, 어디에서나 접근할 수 있다. 
```

**싱글턴이 왜 문제라는 거지?** - 짧게 놓고 보면 큰 문제는 없지만, 길게 놓고 보면 비용을 지부랗게 된다. 꼭 필요하지 않은 곳에 싱글턴을 사용하면 다음과 같은 문제가 발생한다.

- **알고보니 전역 변수**
    - **전역 변수는 코드를 이해하기 어렵게 한다.**
    - **전역 변수는 커플링을 조장한다** - 인스턴스에 대한 접근을 통제함으로써 커플링을 통제할 수 있다.
    - **전역 변수는 멀티스레딩 같은 동시성 프로그래밍에 알맞지 않다** - 멀티스레딩을 최대한 활용하지 못하더ㅏ도 최소한 멀티스레딩 방식에 맞게 코드를 만들어야 한다. 무엇인가를 전역으로 만들면 모든 스레드가 보고 수정할 수 있는 메모리 영역이 생기는 것이다. 다른 스레드가 전역 데이터에 무슨 작업을 하는지 모를 때도 있는데, 이러다 보면 교착상태, 경쟁 상태 등 찾기 어려운 스레드 동기화 버그가 생기기 쉽다.
    
    그럼 전역 상태 없이 게임 아키텍처를 만드려면 어떻게 할까? 싱글턴은 치료제보다는 진정제에 가깝다. 어느 하나도 싱글턴 패턴으로 해결 할 수 없다. 뭐니 뭐니 해도 싱글턴 패턴이 바로 클래스로 캡슐화된 전역 상태이기 때문이다. 
    
- **싱글턴은 문제가 하나뿐일 때도 두 가지 문제를 풀려 든다**
    
    인스턴스를 하나로 강제하고 싶을 뿐, 전역 접근을 원하지 않는다면? 반대로 클래스를 전역에서 접근하고 싶지만 인스턴스 수는 여러 개일 수 있다면? (‘한 개의 인스턴스’와 ‘전역 접근’ 중 보통 ‘전역 접근’이 싱글턴 패턴을 선택하는 이유다.)
    
- **게으른 초기화는 제어할 수가 없다**
    
    가상 메모리도 사용할 수 있고 성능 요구도 심하지 않은 데스크톱 PC에서는 게으른 초기화가 괜찮은 기법이지만, 게임은 다르다. 시스템을 초기화 할 때 메모리 할당, 리소스 로딩 등 할 일이 많다 보니 시간이 꽤 걸릴 수 있다. 청므 소리를 재생할 때 게으른 초기화를 하게 만들면 전투 도중 초기화가 시작되는 바람에 화면 프레임이 떨어지고 버벅거릴 수 있다. 게임에서는 메모리 단편화를 막기 위해 힙에 메모리를 할당하는 방식을 세밀하게 제어하는 게 보통이다. 오디오 시스템이 초기화될 때 상당한 메모리를 힙에 할당하면, 힙 어디에 메모리를 할당할지를 제어할 수 있도록 적절한 초기화 시점을 찾아야 한다. 게으른 초기화 문제를 해결하기 위한 코드를 보자.
    
    ```cpp
    class FileSystem
    {
    public:
    	static FileSystem& instance() { return instance_; }
    	
    private:
    	FileSystem() {}
    	
    	static FileSystem instance_;
    };
    ```
    
    이러면 게으른 초기화 문제를 해결할 수 있지만, 싱글턴이 그냥 전역 변수보다 나은 점을 몇 개 포기해야 한다. 정적 인스턴스를 사용하면 다형성을 사용할 수 없다. 클래스는 정적 객체 초기화 시점에 생성된다. 인스턴스가 필요 없어도 메모리를 해제할 수 없다. 싱글턴 대신 단순한 정적 클래스를 하나 만든 셈이다.


**대안** - 싱글턴을 안 쓰는 것에 대한 대안

**클래스가 꼭 필요한가?** - 이 책의 저자가 게임 코드에서 본 싱글턴 클래스 중에는 객체 관리용으로 존재하는 ‘관리자(manager)’가 많았다고 한다. 관리 클래스가 필요할 때도 있지만, OOP를 제대로 이해하지 못해 만드는 경우도 많다. (…) 관리자 인스턴스를 없애는 코드를 보자.

```cpp
class Bullet
{
public:
	Bullet(int x, int y) : x(x_), y(y_) {}
	
	bool isOnScreen()
	{
		return x_ >= 0 && x_ < SCREEN_WIDTH && y_ >= 0 && y_ < SCREEN_HEIGHT;
	}
	
	void move() { x_ += 5; }
	
private:
	int x_;
	int y_;	
};
```

서툴게 만든 싱글턴은 다른 클래스에 기능을 더해주는 ‘도우미’인 경우가 많다. 가능하다면 도우미 클래스에 있던 작동 코드를 모두 원래 클래스에 옮기자. 객체가 스스로를 챙기게 하는 것이 바로 OOP다. 

**오직 한 개의 클래스 인스턴스만 갖도록 보장하기** - 클래스 인스턴스를 하나만 있도록 보장하는 건 중요한데, 그렇다고 전역으로 어디서나 접근할 수 있게 하는 건 아닐 수 있다. 이럴 때 전역에서 누구나 접근할 수 있게 만들면 구조가 취약해진다.  전역 접근 없이 클래스 인스턴스만 한 개로 보장할 수 있는 방법이 있다. 

```cpp
class FileSystem
{
public:
	FileSystem()
	{
		assert(!instantiated_); // assert : 디버깅 모드에서 오류가 생기면 치명적일 곳에 넣는 오류 검출용 함수
		instactiated_ = true;
	}
	
	~FileSystem() { instanced_ = false; }
	
private:
	static bool instantiated_;
};

// 이 클래스는 어디서나 인스턴스를 생성할 수 있지만, 둘 이상 되는 순간 assert 함수에 걸린다.
// 단일 인스턴스는 보장하지만 클래스를 어떻게 사용할지에 대해서는 강제하지 않는다.
// 다만 싱글턴은 클래스 문법을 활용해 컴파일 시간에 단일 인스턴스를 보장하는 데 반해, 이 방식에서는 런타임에 인스턴스 개수를 확인한다는 게 단점이다.
```

**인스턴스에 쉽게 접근하기** - 쉬운 접근성은 싱글턴을 선택하는 가장 큰 이유다. 이런 편리함에는 원치 않는 곳에서도 쉽게 접근할 수 있다는 비용이 따른다. 변수는 작업 가능한 선에서 적은 범위로 노출하는 게 일반적으로 좋다. 변수가 노출된 범위가 적을수록 코드를 볼 때 머릿속에 담아둬야 할 범위가 줄어든다. 전역에서 접근하는 것 말고 객체에 접근할 수 있는 다른 방법을 알아보자.

- **넘겨주가** - 객체를 필요로 하는 함수에 인수로 넘겨주는 게 가장 쉬우면서도 최선인 경우가 많다. 반면 어떤 객체는 메서드 시그니처에 포함되지 않는다. 다른 방법을 찾아보자.
- **상위 클래스로부터 얻기** - 게임 내 객체가 상속받는 GameObject라는 상위 클래스가 있다고 해보자. 많은 클래스에서 GameObject 상위 클래스에 접근할 수 있다. 이 점을 활용하면 아래와 같이 만들 수 있다.

```cpp
class GameObject
{
protected:
	Log& getLog() { return log_; }
	
private:
	static Log& log_;
};

class Enemy : public GameObject
{
	void doSomething() { getLob.write("I can log!"); }
};
// 이러면 GameObject를 상속받은 코드에서만 getLog()를 통해 로그 객체에 접근할 수 있다.
```

- **이미 전역인 객체로부터 얻기** - 전역 상태를 모두 제거하기란 너무 이상적이다. 결국 Game이나 World같이 전체 게임 상태를 관리하는 전역 객체와 커플링되어 잇기 마련이다. 기존 전역 객체에 빌붙으면 전역 클래스 개수를 줄일 수 있다. Log, FileSystem, Audio Player를 각각 싱글턴으로 만드는 대신 이렇게 해보자.

```cpp
class Game
{
public:
	static Game& instance() { return instance_; }
	
	Log& getLog() { return *log_; }
	FileSystem& getFileSystem() { return *fileSystem_; }
	AudioPlayer& getAudioPlayer() { return *audioPlayer_; }
	
	// log_ 등을 설정하는 함수들...

private:
	static Game instance_;
	Log *log_;
	FileSystem *fileSystem_;
	AudioPlayer *audioPlayer_;
};

// 이제 Game 클래스 하나만 전역에서 접근할 수 있다. 다른 시스템에 접근하려면
Game::instance().getAudioPlayer().play(VERY_LOUD_BANG); // 이렇게 호출하면 된다.

// 나중에 Game 인스턴스를 여러 개 지원하도록 구조를 바꿔도 Log, FileSystem, AudioPlayer는 영향을 받지 않는다.
// 더 많은 코드가 Game 클래스에 커플링 된다는 단점은 있지만, 여러 방법을 조합해 해결 할 수 있다.
// 이미 Game 클래스를 알고 있는 코드에서는 AudioPlayer를 Game 클래스로부터 받아서 쓰면 되고, 모르는 코드에서는 ㄴ머겨주가나 상위 클래스로부터 얻어서 접근하면 된다.
```

- **서비스 중개자로부터 얻기** - 여러 객체에 대한 전역 접근을 제공하는 용도로만 사용하는 클래스를 따로 정의하는 방법도 있다. 16장에서 살펴볼 것이다.


## ch7 상태(24.09.25~10.01)

이번 장에서는 상태 패턴(유한 상태 기계, 계층형 상태 기계, 푸시다운 오토마타 등)을 다룬다. 

**추억의 게임 만들기** - 사용자 입력에 따라 점프 하는 코드를 추가할 때, 점프 중 점프를 막도록 isJumping_ = true 이런 식의 코드를 계속 추가하면 플래그 변수를 끝도 없이 추가해야 한다. (게임 프로그래밍 올인원 강의랑 2DGP 학교 수업에서 배운 상태 패턴 내용이다!!! 절대 불리언으로 상태를 관리하지 말 것) 플래그 변수를 계속 늘려가면, 버그에 파묻혀 구현을 못 끝낼 것이다. 

**FSM이 우리를 구원하리라** - 주인공이 할 수 있는 동작(서 있기, 점프, 엎드리기, 내려찍기) 등을 칸에 그리고 어떤 버튼을 눌렀을 때 상태가 바뀐다면 이전 상태에서 다음 샅애로 도착하는 화살표를 그린 뒤 눌렀던 버튼을 선에 적는다. 

상태 기계를 그린 플로차트
![image](https://github.com/user-attachments/assets/a15193d4-8163-4cb7-9206-5a78d4e65a9e)


참고) 2DGP Drill09에서 했던 상태 다이어그램 그림이다.
![image](https://github.com/user-attachments/assets/925b25c5-f42d-4ba6-836e-3fdc9697b276)


유한 상태 기계(FSM)은 컴퓨터 과학 분야 중의 하나인 오토마타 이론에서 나왔다. 요점은 이렇다.

- 가질 수 있는 ‘상태’가 한정된다. 예제에서는 서기, 점프, 엎드리기, 내려찍기가 있다.
- 한 번에  ‘한 가지’ 상태만 될 수 있다. 주인공은 점프와 동시에 서 있을 수가 없다. 동시에 두 가지 상태가 되지 못하도록 막는 게 FSM을 쓰는 이유 중 하나다.
- ‘입력’이나 ‘이벤트’가 기계에 전달된다. 예제로 치면 버튼 누르기와 버튼 떼기가 이에 해당한다.
- 각 상태에는 입력에 따라 다음 상태로 바뀌는 ‘전이transition’가 있다. 입력이 들어왔을 때 현재 상태에 해당하는 전이가 있다면 전이가 가리키는 다음 상태로 변경한다.

순수하게 상태만 놓고 보면 상태, 입력, 전이가 FSM의 전부다. 

**열거형과 다중 선택문** - FSM 상태를 열거형으로 정의할 수 있다. 플래그 변수 여러 개 대신 state_ 필드 하나만 있으면 된다. 

```cpp
enum state
{
	STATE_STANDING,
	STATE_JUMPING,
	STATE_DUCKING,
	STATE_DIVING
};

void Heroine::handleInput(Input input)
{
	switch (state_)
	{
		case STATE_STANDING:
			if (input == PRESS_B)
				{
					state_ = STATE_JUMPING;
					yVelocity = JUMP_VELOCITY;
					setGraphics(IMAGE_JUMP);
				}
			else if (input == PRESS_DOWN)
				{
					state_ = STATE_DUCKING;
					setGraphics(IMAGE_DUCK);
				}
			break;
		
		case STATE_JUMPING:
			if (input == PRESS_DOWN)
				{
					state_ = STATE_DIVING;
					setGraphics(IMAGE_DIVE);
				}
			break;
			
		case STATE_DUCKING:
			if (input == RELEASE_DOWN)
				{
					state_ = STATE_STANDING;
					setGraphics(IMAGE_STAND);
				}
			break;
	}
}
```

열거형은 상태 기계를 구현하는 가장 간단한 방법이고, 이 정도만으로 충분할 때도 꽤 있다. 

열거형 만으로는 부족할 수 있는데, 이동을 구현하되 엎드려 있으면 기가 모여서 놓는 순간 특수 공격을 쓸 수 있게 만든다고 해보자. 엎드려서 기를 모으는 시간을 기록해야 한다. 이를 위해 Herion에 chargeTime_ 필드를 추가해보자. 매 프레임마다 호출되는 update() 메서드는 이미 있었다고 치고, 다음 코드를 추가해보자.

```cpp
void Herion::update()
{
	if (state_ == STATE_DUCKING)
	{
		chargeTime_++;
		if (chargeTime_ > MAX_CHARGE)
		{
			superBomb();			
		}
	}
}

void Herion::HandleInput(Input input)
{
	switch (state_)
	{
		case STATE_STANDING:
			if (input == PRESS_DOWN)  // 엎드릴 때마다 시간 초기화 해야 한다. 
			{
				state_ == STANDING_DUCKING;
				chargeTime_ = 0;
				SetGraphics(IMAGE_DUCK);
			}
			
			// 다른 입력 처리
			break;
			
			// 다른 상태 처리...
	}
}
```

기 모으기 공격을 추가하기 위해 함수 두 개를 수정하고 엎드리기 상태에서만 의미 있는 chargeTime_ 필드를 Herion에 추가해야 했다. 이 보다는 모든 코드와 데이터를 한곳에 모아둘 수 있는 게 낫다. GoF가 나설 차례이다!

**상태패턴** - GoF가 설명한 상태 패턴을 Herion 클래스에 적용해보면 다음과 같다.

- **상태 인터페이스** - 상태 인터페이스부터 정의하자. 상태에 의존하는 모든 코드(다중 선택문에 있던 동작)를 인터페이스의 가상 메서드로 만든다. 예제에서는 handleInput()과 update()이다

```cpp
class HeroinState
{
public:
	virtual ~HeroinState() {}
	virtual void handleInput(Heroine& herione, Input input) {]
	virtual void update(Herione& heroine) {}
};
```

- **상태별 클래스 만들기** - 상태별로 인터페이스를 구현하는 클래스도 정의한다. 메서드에는 정해진 상태가 되었을 때 주인공이 어떤 행동을 할지를 정의한다. 다중 선택문에 있던 case 별로 클래스를 만들어 코드를 옮기면 된다.

```cpp
class DuckingState : public HeroinState
{
public:
	DuckingState() : chargeTime_(0) {]
	
	virtual void handleInput(Herion& herione, Input input)
	{
		if (input == RELEASE_DOWN)
		{
			// 일어선 상태로 바꾼다...
			herione.setGraphics(IMAGE_STAND);
		}
	}
	
	virtual void update(Herione& heroine)
	{
		chargeTime_++;
		if (chargeTime_ > MAX_CHARGE)
		{
			herione.superBomb();
		}
	}
	
private:
	int chargeTime_;  
	// 변수를 Herione에서 DuckingState 클래스로 옮겼다!
	// chargeTime_ 변수는 엎드리기 상태에서만 의미 있다는 점을 분명하게 보여준다
}
```

- **동작을 상태에 위임하기** - 이번에는 Herion 클래스에 자신의 현재 상태 객체 포인터를 추가해, 거대한 다중 선택문은 제거하고 대신 상태 객체에 위임한다.

```cpp
class Herion
{
public:
	virtual void handleInput(Input input)
	{
		state_->handleInput(*this, input);
	}
	
	virtual void update() { state_ ->update(*this); }

	// 다른 메서드들...
	
private:
	HerionState *state_;
	// 상태를 바꾸려면 state_ 포인터에 HerionState를 상속받는 객체를 할당하기만 하면 된다. 
}
```

**상태 객체는 어디에 둬야 할까?** - 상태를 바꾸려면 state_에 새로운 상태 객체를 할당해야 한다. 상태 패턴은 열거형이 아닌 클래스를 쓰기 때문에 포인터에 저장할 실제 인스턴스가 필요하다. 두 가지 방법을 알아보자.

- **정적 객체** - 상태 객체에 필드(멤버 변수)가 따로 없다면 가상 메서드 호출에 필요한 vtable 포인터만 있는 셈이다. 이 경우 모든 인스턴스가 같기 때문에 인스턴스는 하나만 있으면 된다. 정적 인스턴스를 원하는 곳에 두면 된다. 특별히 다른 곳이 없다면 상위 클래스에 두자
    
    ```cpp
    class HerionState
    {
    public:
    	static StadingState standing;
    	static DuckingState ducking;
    	static JumpingState jumping;
    	static DivingState diving;
    	// 다른 코드들...
    };
    
    // 각각의 정적 변수가 게임에서 사용하는 상태 인스턴스다. 서 있는 상태에서 점프하게 하려면 이렇게 한다.
    if (input == PRESS_B)
    {
    	heroine.state_ = &HerioineState::jumping;
    	herione.setGraphics(IMAGE_JUMP);
    }
    ```
    

- **상태 객체 만들기** - 정적 객체만으로 부족할 떄도 있다. 엎드리기 상태에서 chargeTime_ 필드가 있는데 이 값이 주인공마다 다르니 정적 객체로 만들 수 없다. 주인공이 둘 이상일 때 문제가 된다. 이럴 때는 전이할 때마다 상태 객체를 만들어야 한다. 이러면 FSM이 상태별로 인스턴스를 갖게 된다. 새로 상태를 할당했기 떄문에 이전 상태를 해제해야 한다. 상태를 바꾸는 코드가 현재 상태 메서드에 있기 때문에 삭제할 때 this를 스스로 지우지 않도록 주의해야 한다. 이를 위해 handleInput()에서 상태가 바뀔 때에만 새로운 상태를 반환하고, 밖에서는 반환 값에 따라 예전 상태를 삭제하고 새로운 상태를 저장하도록 바꿔보자.
    
    ```cpp
    void Herione::handleInput(Input input)
    {
    	HeroineState* state = state_->handleInput(*this, input);
    	
    	if (state != NULL)
    	{
    		delete state_;
    		state_ = state;
    	}
    }
    
    // handleInput 메서드가 새로운 상태를 반환하지 않는다면 현재 상태를 삭제하지 않는다.
    // 서 있기 상태에서 엎드리기 상태로 전이하려면 새로운 인스턴스를 생성해 반환한다. 
    
    HeroineState* StandingState::handleInput(Heroine& heroine, Input input)
    {
    	if (input == PRESS_DOWN)
    	{
    		// 다른 코드들...
    		return new DukingState();
    	}
    	
    	// 지금 상태를 유지한다.
    	return NULL;
    }
    ```
    

→ 이 책의 저자는 가능하다면 매번 상태 객체를 할당하기 위해 메모리와 CPU를 낭비하지 않아도 되는 정적 상태를 쓰는 편이다. 지금부터는 상태 패턴을 ‘상태스럽게’ 만드는 방법을 살펴본다.

**입장과 퇴장** - 상태 패턴의 목표는 같은 상태에 대한 모든 동작과 데이터를 클래스 하나에 캡슐화하는 것이다. 이런 면에서 예제는 아직 부족한 면이 있다. 주인공은 상태를 변경하면서 주인공의 스프라이트도 바꾼다. 지금까지는 이전 상태에서 스프라이트를 변경했다. (e.g. [엎드리기 → 서있기] - 엎드리기 상태에서 주인공 이미지 변경) 이렇게 하는 것보다 **상태에서 그래픽까지 제어**하는 게 바람직하다. 이를 위해 **입장 기능**을 추가한다.

```cpp
class StandingState : public HeroineState
{
public:
	virtual void enter(Heroine& heroine) { heroine.SetGraphics(IMAGE_STAND); }
	// ect...
};

// enter 함수를 호출하도록 상태 변경 코드를 수정한다.
void Heroine::handleInput(Input input)
{
	HeroineState* state = state_->handleInput(*this, input);
	
	if (state)
	{
		delete state_;
		state_ = state;
	}
	
	// 새로운 상태의 입장 함수를 호출한다.
	state_->enter(*this);
}

// 이제 엎드리기 코드를 더 단순하게 만들 수 있다.
HeroineState* DuckingState::handleInput(Heroine& heroine, Input input)
{
	if (input == RELEASE_DOWN)
	{
		return new StandingState();
	}
	// ect...
}
```

서기 상태로 변경하면 서기 상태가 알아서 그래픽까지 챙긴다. 이제 제대로 캡슐화 되었다. 그 전 상태와 상관없이 항상 같은 입장 코드가 실행된다는 것도 장점이다. **퇴장코드(상태가 새로운 상태로 교체되기 직전에 호출되는)**도 이런 식으로 활용할 수 있다.

**단점은?** - 상태 기계는 엄격하게 제한된 구조를 강제함으로써 복잡하게 얽힌 코드를 정리할 수 있게 해준다. FSM에는 미리 정해 놓은 여러 상태와 현재 상태 하나, 하드코딩 되어 잇는 전이만이 존재한다. 인공지능 같이 복잡한 곳에 적용하는 데 한계가 있다. 다행이 몇 가지 방법이 있다.

**병행 상태 기계** - 주인공이 총을 들어도 달리기, 점프, 엎드리기 등의 동작을 모두 할 수 있어야 한다. 동시에 총도 쏠 수 있어야 한다. FSM 방식을 고수한다면 모든 상태를 서기, 무장 상태로 서기, 점프, 무장 상태로 점프 같이 무장과 비무장에 맞춰 두 개씩 만들어야 한다. 무기를 추가할 수록 조합이 폭발적으로 늘어난다. 해결 방법은 간단한데, 상태 기계를 둘로 나누면 된다. 무엇을 하는가에 대한 상태 기계는 그대로 두고, 무엇을 들고 있는가에 대한 상태 기게를 따로 정의한다. Heroine 클래스는 이들 상태를 **각각** 참조한다.

```cpp
class Heroine
{
	// ect...
	
private:
	HeroineState* state_;
	HeroineState* equipment_;
};

// Heroine에서 입력을 상태에 위임할 때에는 입력을 상태 기계 양쪽에 다 전달한다.
void Heroine::handleInput(Input input)
{
	state_->handleInput(*this, input);
	equipment_->handleInput(*this, input);
}
```

각각의 상태 기계는 입력에 따라 동작을 실행하고 독립적으로 상태를 변경할 수 있다. 두 상태 기계가 서로 연관이 없다면 이 방법이 잘 맞는다. 현실적으로 점프 도중 총을 못 쏘거나, 무장한 상태에서는 내려찍기를 못한다든가 하는 식으로 복수의 상태 기계가 상호작용해야 할 수도 있다. 이를 위해 어떤 상태 코드에서 다른 상태 기계의 상태가 무엇인지를 검사한느 지저분한 ㅗ드를 만들 일이 생길 수도 있다. 

**계층형 상태 기계** - 주인공 동작을 만들다 보면 서기, 걷기, 달리기, 미끄러지기 등과 같이 비슷한 상태가 많이 생긴다. 이들 상태에선 모두 B를 누르면 점프하고, 아래 버튼을 누르면 엎드려야 한다. 단순한 상태 기계 구현에서는 이런 코드를 모든 상태마다 중복해 넣어야 한다. 그보다는 한 번만 구현하고 다른 상태에서 재사용하는 게 낫다. 상태 기계가 아니라 객체지향 코드라고 생각해보면, 상속으로 여러 상태가 코드를 공유할 수 있다. 일단 ‘땅 위에 있는’ 상태 클래스를 정의해 처리하고 서기, 걷기, 달리기, 미끄러지기는 이 클래스를 상속받아 고유 동작을 추가하면 된다. 이런 구조를 **계층형 상태 기계**라고 한다. 어떤 상태는 **상위 상태superstate**를 가질 수 있고, 그 경우 그 상태 자신은 **하위 상태substate**가 된다. 이벤트가 들어올 때 하위 상태에서 처리하지 않으면 상위 상태로 넘어간다. 이는 상속받은 메서드를 오버라이드하는 것과 같다. 예제 FSM을 상태 패턴으로 만들면 클래스 상속으로 계층을 구현할 수 있다. 상위 상태용 클래스를 하나 정의하고 하위 상태도 정의해보자.

```cpp
class OnGroundState : public HeroineState
{
public:
	virtual void handleInput(Heroine& heroine, Input input)
	{
		if (input == PRESS_B)
		{
			// 점프...
		}
		else if (input == PRESS_DOWN)
		{
			// 엎드리기...
		}
	}
};

class DuckingState : public OngroundState
{
public:
	virtual void handleInput(Heroine& heroine, Input input)
{
		if (input == REALEASE_DOWN)
		{
			// 서기...
		}
		else 
		{
			// 따로 입력을 처리하지 않고 상위 상태로 보낸다.
			OnGroundState::handleInput(heroine, input);
		}
	}
}
```

**푸시다운 오토마타** - 상태 스택을 활용하여 FSM을 확장하는 다른 방법도 있다. FSM은 **이력history**이 없다는 문제가 있다. 현재 상태는 알 수 있지만 직전 상태는 따로 저장하지 않아 쉽게 돌아갈 수 없다. 예를 들어 총을 쏜 상태를 마치면 다시 원래 상태로 돌아가야 하는데 서기, 달리기, 점프, 엎드리기 중 어떤 상태로 돌아갈지 알 수 없는 것이다. 일반적인 FSM에서는 이전 상태를 알 수 없어 이를 알기 위해서 서 있는 상태에서 총 쏘기, 달려가면서 총 쏘기 등 상태마다 새로운 상태를 만들어야 한다. 이 것보다 총 쏘기 전 상태를 **저장해놨다가** 나중에 불러와 써먹는 게 훨씬 낫다. 바로 **푸시다운 오토마타**를 사용하는 것이다. FSM이 한 개의 상태를 포인터로 관리했다면, 푸시다운 오토마타는 상태를 스택으로 관리한다. FSM은 이전 상태를 덮어쓰고 새로운 상태로 전이하는 방식이었다. 

![image](https://github.com/user-attachments/assets/8c72d10a-40ab-4e98-b406-c3d13c1fb31e)

이렇게 총쏘기 상태를 구현할 수 있다. 어떤 상태에서든 발사 버튼을 누르면 총 쏘기 상태를 스택에 넣는다. 총 쏘기 애니메이션이 끝나고 총 쏘기 상태를 스택에서 빼면, 푸시다운 오토마타가 알아서 이전 상태로 보내준다. 

**얼마나 유용한가?** - FSM의 확장판이 있지만 그래도 한계가 있다. 요즘 게임 AI는 행동 트리나 계획 시스템을 더 많이 쓰는 추세다. FSM은 다음 경우에 사용하면 좋다.

- 내부 상태에 따라 객체 동작이 바뀔 때
- 이런 상태가 그다지 많은 선택지로 분명하게 구분될 수 있을 때
- 객체가 입력이나 이벤트에 따라 반응할 때

게임에서는 FSM이 AI에 사용되는 걸로 가장 잘 알려져 있지만, 입력 처리나 메뉴 화면 전환, 문자 해석, 네트워크 프로토콜, 비동기 동작을 구현하는 데에도 많이 사용되고 있다.


