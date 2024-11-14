---
title: "[SERVER] 멀티 쓰레드 프로그래밍"
date: 2024-09-24 12:50:00 +09:00 
categories: SERVER
author_profile: false
toc : true
toc_sticky: true
---

## 서버 OT(24.09.24)

**게임 서버의 종류**

- Web Server(aka. HTTP Server) - 테이크아웃 포장 전문 식당. 손님이 음식을 받아서 떠나면 그 이후론 연락이 끊긴다. 질의/응답 형태.
- Game Server(aka. TCP Server, Binary Srver, Stateful Server...) - 일반 식당. 서빙 직원이 와서 손님에게 물어볼 수도 있고, 손님이 추가 주문을 하기도 하고. 실시간 Interaction이 있다!

**Web Server** - (e.x. 클래시 오브 클랜). 드물게 정보를 요청/갱신한다. 실시간 Interaction이 필요하지 않다. 식당에서 손님한테 먼저 접근할 일은 없다(물 따라 드릴까요?). 주문 후 손님이 바로 떠나면, 손님의 상태를 당분간 잊고 지낸다(Stateless). 단순히 게임에 국한되지 않고, 웹 서비스를 만드는 데 사용(구글, 아마존, 네이버 등). 처음부터 만드는 경우는 사실상 없고, 프레임 워크를 하나 골라서 사용(ASP.NET(C#), Spring(Java), Node JS(JavaScript),…). 

**Game Server** - (e.x. 스타 크래프트, 월드 오브 워). 요청/갱신 횟수가 많다. 실시간 Interaction이 필요하다. 언제라도 직원이 손님한테 접근 가능해야 한다. 손님이 식당에 머무는 동안, 손님의 상태를 보며 최상의 서비스를 제공한다(Stateful). 제작은? 게임/장르에 따라 요구 사항이 너무 다르다(최적의 프레임워크라는 것이 존재하기 애매). 

| 고려할 대상 | 게임 서버의 유사성 |
| --- | --- |
| 손님 한도(몇 명까지 받을 수 있는지) | 최대 동시 접속자 |
| 한 방에 들어갈 수 있는 손님의 일행 한도(인테리어) | 게임 장르 및 채널링 |
| 직원 역할 구분(겸직 가능) | 게임 로직(요리사), 네트워크(서빙), DB(결제) |
| 직원은 몇 명을 둘지? | 쓰레드 개수 |
| 요리사/서빙/결제 직원 비율을 어떻게? | 쓰레드 모델 |
| 주문은 어떻게 받을까?(손님이 불러서? 벨?) | 네트워크 모델 |
| 손님이 기다릴 수 있는 시간 한도(패스트 푸드? 고급?) | 반응성(FPS, MMORPG…) |
| 장부 및 결제는 어떻게? | 데이터베이스 |

cf) 서버를 직접 구현하면 4~5년 걸림, 언리얼 엔진에서 데디 서버 사용하면 4~5년 걸릴 거 1년 걸린다.

FPS 게임(like 배그)의 경우 데디 서버(근데 방이 많다! MMO 보다 쉽지만 플랫폼 작업에 어려움), MMORPG는 서버 구축하는데 몇 년 걸림. **데디 서버는 꼭 공부하는 게 좋다!!!**

---

## 멀티쓰레드 입문(24.09.27, 29)

💥면접에 항상 나오는 질문(기본) - 프로세스와 쓰레드의 차이 ← 멀티 쓰레드에 대한 개념이 있는지 확인


그림판, 메모장, 게임 등 프로그램들을 cpu가 하나씩 번갈아가며 실행하는데 너무 빨라서 동시에 실행되는 것처럼 보인다.

멀티 코어를 이용하면 CPU는 하나지만  동시에 다수의 쓰레드들을 실행할 수 있다.

cf) 서버 전용 운영체제가 따로 있다(서버만 집중적으로 꼭 필요한 것들만 실행).


쓰레드(직원)가 있고 CPU 코어(실행권)가 배치되어 일을 처리하도록 한다. 

![image](https://github.com/user-attachments/assets/c02df6a0-2d30-41f8-893d-6f630cdf3d6e)


쓰레드를 여러개 배치해서 분담시킨다.

![image](https://github.com/user-attachments/assets/61241acc-d5cc-4cf9-b83b-ee0a2ca1ab8a)

cf) 배그는 단일 쓰레드, MMO는 무조건 멀티 쓰레드로(처리할 게 너무 많아서 - 직원 한 명이 모든 일을 다 처리하지 못함)

그렇다고 쓰레드가 다다익선인가? 그건 아니다. 이론적으로는 잘 배치될 것 같지만, 어느 작업에 몰려서 다른 작업을 제대로 처리하지 못하는 싱글 쓰레드 만도 못한 문제가 발생한다.

![image](https://github.com/user-attachments/assets/18fe1795-5999-4a39-b903-a08cb1b0955c) | ![image](https://github.com/user-attachments/assets/18300934-9cf7-4cce-8532-a1f840181740)


문맥 교환(context switching)으로 왔다갔다 하는 비용이 상당하다.

![image](https://github.com/user-attachments/assets/31b79951-cfa0-48d3-ac4b-e598984f5544)

---

## 쓰레드 생성(24.09.30)

실습 코드 [쓰레드 생성](https://github.com/chaeeun-dev/Server/commit/5515179eb2fb9734efdffb1361c9d4f9cbeec439)

---

## 쓰레드 보충 공부(24.10.03)

[모두의 코드 씹어먹는 C++ thread](https://modoocode.com/269)

프로세스 - 운영체제에서 실행되는 프로그램의 최소 단위, 보통 한 개의 프로그램을 가리킬 때 한 개의 프로세스를 의미하는 경우가 많다.

프로세스는 컴퓨터의 두뇌인 CPU 코어에서 실행된다. 옛날에는 1코어가 대부분이어서 한 번에 한 가지 연산 밖에 못했다. 동시에 여러 일을 할 수 있었던 이유는 **Context switching(문맥교환)** 기술 덕분이다. 

![image](https://github.com/user-attachments/assets/d2f453ce-6e32-4056-a6ea-95b90e26be3b)

프로그램이 하나 실행되고, 다른 프로그램으로 스위칭 되는 과정을 거친다. 조금씩 차례를 돌며 실행시키는 것이다. CPU는 처리할 명령어를 실행할 뿐, 어떤 프로그램을 실행시키고, 얼마 동안 실행시키고, 어떤 프로그램으로 스위치 할지는 **운영체제의 스케줄러**가 결정하는 것이다.

쓰레드 - CPU 코어에서 돌아가는 프로그램 단위를 **쓰레드**라고 한다. CPU 코어 하나에서 한 번에 한 개의 쓰레드만 실행할 수 있다. 한 개의 프로세스는 최소 한 개의 쓰레드로 이루어져 있으며, 여러 개의 쓰레드로 구성될 수 있다. 이렇게 여러 개의 쓰레드로 구성된 프로그램을 **멀티 쓰레드 프로그램**이라고 한다. 

쓰레드와 프로세스의 가장 큰 차이점 - 쓰레드는 서로 같은 메모리를 공유하고, 프로세스들은 서로 메모리를 공유하지 않는다.

![image](https://github.com/user-attachments/assets/a5eeeccc-b171-426e-896e-54c622f7fc61)

여태까지 작성했던 프로그램들은 모두 한 개의 쓰레드로 구성된 프로그램이다. 

**CPU의 코어는 한 개가 아니다.** - CPU의 여러 코어에 각기 다른 쓰레드들이 들어가 동시에 여러 개의 쓰레드들을 효율적으로 실행할 수 있다. 

![image](https://github.com/user-attachments/assets/cbbaf858-d53e-44c3-a9a4-29b1e3815007)


**join**은 해당하는 쓰레드들이 실행을 종료하면 리턴하는 함수이다. **detach**는 해당 쓰레드를 실행시킨 후 잊어버리는 것이다. 대신 쓰레드는 알아서 백그라운드에서 돌아가게 된다. 

---

## 캐시와 CPU 파이프라인(24.10.03)

CPU는 연산을 담당하고, RAM은 데이터 저장을 담당한다. 둘의 거리가 꽤 먼데 데이터를 계속 주고 받는 건 낭비가 될 것이다. 그래서 **캐시**를 사용한다. 

**캐시 철학**
1. TEMPORAL LOCALITY - 시간적으로 보면, 최근에 접근한 메모리는 또 접근할 확률이 높다.
2. SPATIAL LOCALITY - 공간적으로 보면, 접근한 메모리 근처에 있는 메모리에 접근할 확률이 높다. 

---

### 실습1 - 캐시

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <windows.h>

int buffer[10000][10000];

int main()
{
	std::memset(buffer, 0, sizeof(buffer));		// 0으로 밀어 넣음

	{
		auto start = GetTickCount64();

		__int64 sum = 0;

		for (int i = 0; i < 10000; i++)
			for (int j = 0; j < 10000; j++)
				sum += buffer[i][j];

		auto end = GetTickCount64();

		std::cout << "Elapsed Tick" << (end - start) << std::endl;

	}

	{
		auto start = GetTickCount64();

		__int64 sum = 0;

		for (int i = 0; i < 10000; i++)
			for (int j = 0; j < 10000; j++)
				sum += buffer[j][i];	// j, i 순서만 바꿈

		auto end = GetTickCount64();

		std::cout << "Elapsed Tick" << (end - start) << std::endl;

	}

	// 결과 
	// Elapsed Tick94
	// Elapsed Tick375
}

```

알고리즘 측면에서는 두 코드가 같은데, 시간이 약 3배 이상 차이가 나는 이유는? 바로 **캐시**때문이다. 첫 번째 코드에서는 순차적으로 접근하기 때문에 캐시 히트가 발생하고, 두 번째 코드에서는 첫 번째, 만 번 뒤, 만 번 뒤... 이렇게 띄엄띄엄 접근하기 때문에 캐시 히트가 일어나지 않는다. 캐시가 제대로 동작한다는 걸 알 수 있는 실습이다.


**4단계 파이프라인**

![image](https://github.com/user-attachments/assets/abfdcdb6-6141-4e96-816d-0eeb5a0b4bde)

1. Fetch - 명령어를 갖고 온다.
2. Decode - 어떤 명령어인지 해석한다.
3. Execute - 명렁어를 실행한다.
4. White-back - 

예) 세탁기, 건조기, 다리미를 사용할 때 세탁물을 하나씩 순서대로 기다리며 처리하면 시간이 너무 오래 걸린다.  그래서 세탁기, 건조기, 다리미가 비어있으면 세탁물을 채우는 식으로 진행한다. 

---

### 실습2 - 컴퓨터 구조 원리 : 파이프라인

CPU가 지시 받을 때 무조건 받은 순서대로 처리하지 않는다. 실행하다보니 더 빠른 것 같으면 자기 마음대로 순서를 바꿔 실행할 수도 있다는 것이다. 


CPU 파이프라인 최적화 코드
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <windows.h>

int x = 0;
int y = 0;
int r1 = 0;
int r2 = 0;

bool ready = false;

void Thread_1()
{
	while (ready == false){ }

	y = 1;	// Store y
	r1 = x;	// Load x
}

void Thread_2()
{
	while (ready == false){ }

	x = 1;	// Store x
	r2 = y;	// Load y
}

int main()
{
	int count = 0;

	while (true)
	{
		ready = false;
		count++;

		x = y = r1 = r2 = 0;

		std::thread t1(Thread_1);
		std::thread t2(Thread_2);

		ready = true;

		t1.join();
		t2.join();

		if (r1 == 0 && r2 == 0)	
			break;
	}

	std::cout << count << std::endl;
}
```

![image](https://github.com/user-attachments/assets/d6912d93-3079-474b-804e-5c5c847b285b)

실습은 진행되는 코드의 순서가 중요한 경우 위험할 수 있다는 것을 보여준다. 멀티 쓰레드 환경에서 정말 끔찍한 상황이 발생할 것이다.


컴파일러 최적화 코드
```cpp
volatile bool ready = false;	// volatile : 코드 최적화를 하지 말아달라는 용도

void Thread_1()
{
	while (ready == false){ }	// 컴파일러가 while (false)로 코드 최적화를 할 수도 있다.

	std::cout << "Yeah!" << std::endl;
}

int main()
{
	std::thread t1(Thread_1);

	std::this_thread::sleep_for(std::chrono::seconds(1));  // 1초 대기

	ready = true;

	t1.join();
}
```

**싱글 쓰레드 환경에서 잘 작동하던 코드가 멀티 쓰레드에서는 먹통이 될 수 있다, 환경에 따라 굉장히 다르게 동작할 수 있다.**

이 부분은 써먹기 보다는 상식이라 배운 것!

---

## 공유 자원(24.10.07)

### 실습1 - 공유 자원의 문제

```cpp
#include <windows.h>
#include <thread>
#include <iostream>
#include <atomic>

// Code X
// Stack X
// Heap O
// Data O

// thread에서 Heap, Data 영역을 조심해야 한다.
// 보자마자 데이터를 공유해서 사용할 수 있는 코드인지 확실하게 판별할 수 있어야 한다! 1초만에.

int sum = 0;	// sum은 Data 영역에 들어간다.

void Add()
{
	for (int i = 0; i < 100'0000; i++)
	{
		sum++;

		// 디스어셈블리 창으로 확인하면
		/*int eax = sum;
		eax = eax + 1;
		sum = eax;*/
		// 이렇게 실행이 된다.

		// why? sum 변수는 RAM만 알고 CPU가 알지 못하기 때문에(멀어서) 레지스터에 넣은 값으로 연산을 수행한다. 
	}
}

void Sub()
{
	for (int i = 0; i < 100'0000; i++)
	{
		sum--;

		/*int eax = sum;
		eax = eax - 1;
		sum = eax;*/
	}
}

int main()
{
	std::thread t1(Add);
	std::thread t2(Sub);

	t1.join();
	t2.join();

	std::cout << "sum == " << sum << std::endl;


}
```

결과는 i가 100일 때 sum == 0, i가 100'0000일 때 sum == 178827 (정말 어이 없는 결과... 컴퓨터한테 뒷통수 맞은 기분이다)

문제의 원인? 중간에 동시다발적으로 끼어들어서 문제 발생

---

### 실습2 - atomic으로 공유 자원 문제 해결

```cpp
#include <windows.h>
#include <thread>
#include <iostream>
#include <atomic>

// Code X
// Stack X
// Heap O
// Data O

// thread에서 Heap, Data 영역을 조심해야 한다.
// 보자마자 데이터를 공유해서 사용할 수 있는 코드인지 확실하게 판별할 수 있어야 한다! 1초만에.

std::atomic<int> sum = 0;

void Add()
{
	for (int i = 0; i < 100'0000; i++)
	{
		// All or Nothing (모두 되거나 아예 안 되거나)
		sum++;
	}
}

void Sub()
{
	for (int i = 0; i < 100'0000; i++)
	{
		sum--;
	}
}

int main()
{
	std::thread t1(Add);
	std::thread t2(Sub);

	t1.join();
	t2.join();

	std::cout << "sum == " << sum << std::endl;
	// sum == 0
}
```
sum을 atomic(클래스)으로 만들었더니 앞의 문제가 해결되었다. 세 단계에 걸쳐 실행됐던 코드가 한 단계로 줄어든 것이다.

why? 앞의 어셈블리어는 inc였다면, atomic으로 실행했을 때 어셈블리어는 locked inc로 아예 다른 명령어가 실행된다.

그렇다고 멀티 스레드에서 모든 정수를 atomic으로 만들면? 안 된다. 일반 정수 연산보다 훨씬 느리다. 

---

### 실습3 - atomic 실습

```cpp
	std::atomic<int> sum = 0;

	// int temp = sum;
	int temp = sum.load();	// 윗줄과 똑같은 뜻, 단지 atomic 타입이라는 힌트를 줄 뿐이다.
	
	// sum = 10;
	sum.store(10);	// 윗줄과 똑같은 뜻, 마찬가지로 atomic이라는 힌트를 준다.

	// 값을 넣을 때 그 이전의 값이 궁금하면
	int temp = sum.exchange(10);	
```

---

### 실습4 - 멀티 스레드에서 Stack, Heap 구분하기

```cpp
void Test()
{
	for (int i = 0; i < 10000; i++)
	{
		int* p = new int();		// Heap이지만 공유해서 사용하지 않으므로 멀티 스레드 상황에서도 크래시가 나지 않는다.

		*p = 100;

		delete p;
	}
}
```

![image](https://github.com/user-attachments/assets/b133a9dc-52e9-474d-847e-7927f5c9868a)

p는 Stack 영역에 있어서 다른 스레드들이 알 수 없다. 그리고 Heap이라도 다른 스레드들과 공유하지 않으면 문제되지 않는다. 문제가 될 수 있는 경우는 예를들어, 전역 변수나 함수 인자로 변수를 넘겨주며 데이터를 공유할 때가 있다.

---

### 메모리 구조

![image](https://github.com/user-attachments/assets/3fca32f7-46e5-4af0-ad57-0495b80b9507)

[TCP SCHOOL 메모리의 구조](https://www.tcpschool.com/c/c_memory_structure)

---

## Lock 기초(24.11.10)

C++에서 lock은 mutex로 사용하면 된다.

Mutex : Multal Exclusive(상호베타적 : 한 번에 하나만 실행)

```cpp
#include <mutex>

std::mutex m;
```

---

### Mutex 개념, 실습

이 코드의 결과는?

1) 20000
2) Crash 발생
3) 이상한 값

```cpp
std::mutex m;	

std::vector<int> v;

void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		v.push_back(i);
	}
}

int main()
{
	std::thread t1(Push);
	std::thread t2(Push);

	t1.join();
	t2.join();

	std::cout << v.size() << std::endl;
}
```

&nbsp;

정답 -  Crash가 발생한다.

설명  - 대부분의 STL 컨테이너들을 멀티쓰레드에서 접근하면 거의 Crash 발생한다.

vector의 특징
- vector는 [][][][][] 이 컨테이너 안의 데이터가 다 차면 이사를 한다. 
- vector는 동적 배열인데 Heap 영역에 데이터가 할당된다. 
-> vector는 Heap 영역에 있어 동시다발적으로 사용될 수 있다. 어떤 영역을 스레드 A와 B가 동시에 사용하고 있을 때 A의 컨테이너가 다 차서 이사를 하며 그 영역을 삭제했는데 B는 그 영역을 사용하고 있기 때문에 크래시가 발생하는 것이다!

&nbsp;

그럼 vector.reserve(1000000)로 이사 횟수를 줄이면 어떻게 될까?

➡️ crash는 발생하지 않지만 20000이 아닌 이상한 값이 나온다. why? 이사는 하지 않지만, 하나의 주소를 여러 곳에서 사용하기 때문에. [][][][] A 스레드가 컨테이너에 값을 쓰고 있는 도중에 B 스레드가 같은 영역에 값을 쓸 수 있어서 값이 이상하게 나오는 것이다(같은 위치에 두 개를 쓰는 상황이 발생 - 덮어 씀).

&nbsp;

결론 - 멀티스레드로 vector 같은 컨테이너를 사용할 때 보호 장치 없이 사용하는 행위는 위험하다. push_back() 함수가 동시 다발적으로 실행되는 것을 막아줘야 한다. 

Push() 함수에 lock(), unlock() 함수를 추가하면 정상적으로 20000이 나오게 된다! 

```cpp
void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		m.lock();

		v.push_back(i);

		m.unlock();
	}
}
```

- lock : 이미 사용중이면 대기하도록 잠금
- unlock : 잠금 풀기(풀지 않으면 무한 대기.. 짝 맞추기 기본!)
➡️ 한 번에 하나씩 상호베타적으로 실행된다.

cf) 스레드가 실행되는 순서는 알 수 없다. 

---

### RAII(Resource Acquisition Is Initialization)

RAII 패턴을 이용해 프로그래머가 직접 lock(), unlock()을 호출하지 않게 할 수 있다.

```cpp
template<typename T>
class LockGuard
{
public:
	LockGuard(T& m) : _mutex(m)
	{
		_mutex.lock();
	}

	~LockGuard()
	{
		_mutex.unlock();
	}

private:
	T& _mutex;
};

void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		LockGuard<std::mutex> lockGuard(m);
		//std::lock_guard<std::mutex> lockGuard(m);

		v.push_back(i);

	}
}
```

표준에서 std::lock_guard() 함수를 제공하니 그대로 사용하면 된다!

unique_lock() : lock_guard()와 비슷한데, 추가적인 기능으로 lock을 미루고 필요할 때 잠글 수 있다. 


💥lock이 없는데 vector에 접근하는 코드를 봤을 때 바로 문제점을 캐치할 수 있어야 한다! 또 lock을 남발하면 싱글 스레드만도 못한 코드를 만들 수 있으니 주의할 것.

---

## 스핀락(24.11.10)

### lock class 구현 실습
Lock을 class로 직접 구현해보자.

이 코드의 결과는?

1) 20000
2) Crash 발생
3) 이상한 값

```cpp
class Lock
{
public:
	void lock()
	{
		while (_flag)
		{
			// Do Nothing
		}

		_flag = true;
	}

	void unlock()
	{
		_flag = false;
	}

private:
	bool _flag = false;
};

Lock m;
std::vector<int> v;

void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		std::lock_guard <Lock> lockGuard(m);

		v.push_back(i);
	}
}
```

&nbsp;

결과)

lock의 역할을 제대로 못하고 있다(이상한 값이 나오거나 vector에 reserve를 하지 않았을 땐 crash가 발생함).

설명)

while문에서 flag가 true가 됐을 때 동시에 값을 쓸 수 있기 때문이다(코드가 두 단계로 나뉜 게 문제 원인 - while문과 flag 값을 변경하는 코드 사이). 즉 상호베타적인 조건이 깨져버렸기 때문이다.

![image](https://github.com/user-attachments/assets/ff0ee502-a2c6-4ed3-8c05-7c35f4c7da3f) | ![image](https://github.com/user-attachments/assets/f0050f38-b069-499c-8484-a0afa14732e3)

---

### CAS (Compare-And-Swap)
두 단계를 한 번에 일어나도록 구현해보자.

```cpp
class Lock
{
public:
	void lock()
	{
		// CAS (Compare-And-Swap)
		// 두 단게를 하나로 합치는 코드

		bool expected = false;
		bool desired = true;

		while (_flag.compare_exchange_strong(OUT expected, desired) == false)
		{
			expected = false;
		}

		// 의사 코드 - 이 코드가 위에서는 한 번에 일어난다.
		/*
		if (_flag == expected)
		{
			_flag = desired;
			return true;
		}
		else
		{
			expected = _flag;
			return false;
		}
		*/

	}

	void unlock()
	{
		_flag = false;
	}

private:
	std::atomic<bool> _flag = false;
};

```

&nbsp;

앞 코드에서 이 두 단계가 있었는데 CAS를 이용해 하나로 합칠 수 있다.

```cpp
	while (_flag)		// 단계 1 - expected가 true면 아무것도 하지 않는다.
	{
		// Do Nothing
	}

	_flag = true;		// 단계 2 - expected가 false면 desired를 true로 바꾼다.
```

---

### 스핀락
lock이 풀렸는지 계속 계속 뺑뺑이를 돌며 확인하면서 풀렸을 때 lock을 획득하는 방법(존버 메타) - e.g. 화장실이 한 개인데 문 앞에서 계속 기다리는 것. 안에 들어있는 사람이 안 나온다면 시간 낭비이다.

```cpp
// SpinLock
while (_flag.compare_exchange_strong(OUT expected, desired) == false)
{
	expected = false;
}

```

&nbsp;

스핀락의 장단점
- 장점 : 아주 짧은 순간 lock 잡고 unlock 할 것이라면, 효율적이다(보통은 lock을 오래 잡는 경우가 많지 않아서 MMO에서 많이 씀).
- 단점 : 계속 확인해야 하기 때문에 CPU 비용이 많이 든다. 한 스레드가 lock을 오래 붙잡고 있는다면 계속 기다려야한다.

---

## 데드락(24.11.10 ~ 11)

데드락(Deadlock, 교착상태) : lock을 했는데 unlock을 하지 않아 무한 대기에 빠진 상태를 말한다.

데드락 예방 방법 - 소멸될 때 unlock을 하도록 락가드를 사용한다. 그런데 이렇게 간단하게 해결되지는 않는다. 예시로 여러 매니저들이 뮤텍스를 복잡하게 늘어지며 사용하기 때문에 문제를 찾기 어렵다.

---

### 데드락 실습

```cpp
class User
{

};

class UserManager
{
public:
	User* GetUser(int id)
	{
		std::unique_lock<std::mutex> guard(_lock);

		if (_users.find(id) == _users.end())
			return nullptr;

		return _users[id];
	}

private:
	std::map<int, User*> _users;
	std::mutex _lock;
};

std::mutex m1;	// Account Manager
std::mutex m2;	// User Manager

void Thread_1()
{
	for (int i = 0; i < 10000; i++)
	{
		std::lock_guard<std::mutex> lockGuard1(m1);
		std::lock_guard<std::mutex> lockGuard2(m2);
		// TODO
	}
}

void Thread_2()
{
	for (int i = 0; i < 10000; i++)
	{
		std::lock_guard<std::mutex> lockGuard1(m2);
		std::lock_guard<std::mutex> lockGuard2(m1);
	}
}

int main()
{
	std::thread t1(Thread_1);
	std::thread t2(Thread_2);

	t1.join();
	t2.join();

	std::cout << "Jobs Done" << std::endl;
}
```

&nbsp;

![image](https://github.com/user-attachments/assets/5f5fa204-fb44-45de-9d89-005a1c7c386d)

상호베타적인 조건이 지켜졌는데 데드락이 발생했다. (사진의 디버깅 중지 옆에 일시 정지 버튼을 누르면 어디서 멈췄는지 확인할 수 있다.) 

why? Thread_1에서 m1을 잡았고, m2를 잡으려고 하는데 Thread_2에서도 m2를 잡고 있고, m1을 잡으려고 하기 때문에 교착상태가 발생하는 것이다. (식사하는 철학자 문제가 생각난다..)

locck을 잡는 순서가 꼬였을 때 문제를 어떻게 해결할까? Thread_2에서 순서를 바꿔주면 해결된다. 

```cpp
void Thread_2()
{
	for (int i = 0; i < 10000; i++)
	{
		std::lock_guard<std::mutex> lockGuard2(m1);
		std::lock_guard<std::mutex> lockGuard1(m2);
	}
}

```
&nbsp;

이렇게 순서를 바꿔주면 "Just Done"이 출력되며 데드락 문제가 해결된다.

근데 이 코드는 Thread가 두 개라서 간단하지만, 규모가 커질수록 순서 맞추는 것도 복잡해진다. (근데 버그 찾기는 쉬움! 찾는 건 쉽지만 고치는 건 상황에 따라 다름.)

또 개발 단계에서는 버그가 일어나지 않지만, 라이브 단계에서 접속자가 늘어났을 때 갑자기 교착상태가 발생할 수도 있다. (이 예제에서도 for문을 10000번으로 설정해서 그렇지, 숫자를 작게 설정하면 교착상태가 발생하지 않음)

cf) lock 구조를 판별하는 방법 중 하나 - 그래프를 그려 cycle이 있는지 판별한다.

➡️결과적으로 lock을 잡는 순서가 꼬인 것이기 때문에, 순서만 잘 맞춰주면 된다! (그치만 완벽하게 처리하는 방법은 없음! 사이클을 그려 런타임에 판별해서 우회할 수는 있음. 그리고 큰 프로젝트가 아닌 이상 데드락은 거의 발생하지 않음.)

---

## 이벤트와 조건 변수(24.11.13)
Manager에 접근할 땐 lock을 걸고 데이터를 가져오는 등의 작업을 수행하면 된다. 

---

### Producer & Consumer 실습 
Producer, Consumer로 일감을 나누는 실습을 해보자. 네트워크 통신을 받아서 일감을 밀어 넣고, 이어 받아서 처리하는 식의 분담 처리 방식이다. 분담하는 이유는 앞에서 lock을 잡기 위해 경합하며 계속 기다리는 게 싱글 스레드만도 못할 수 있기 때문에 멀티 스레드를 쓰는 의미가 없다. 분담을 하면 이 문제를 해결할 수 있다. 

```cpp
std::mutex m;
std::queue<int> q;

void Producer()
{
	while (true)
	{
		std::unique_lock<std::mutex> lock(m);
		q.push(100);

		std::this_thread::sleep_for(100ms);
	}
}

void Consumer()
{
	while (true)
	{
		std::unique_lock<std::mutex> lock(m);
		if (q.empty() == false)
		{
			int data = q.front();
			q.pop();
			std::cout << data << std::endl;
		}
	}
}

int main()
{
	std::thread t1(Producer);
	std::thread t2(Consumer);

	t1.join();
	t2.join();
}
```

```std::this_thread::sleep_for(100ms);``` 이 부분에서 E2486 (사용자 정의 리터럴 연산자가 없습니다.), E0304(인수 목록이 일치하는 함수 템플릿 'std::this_thread::sleep_for'의 인스턴스가 없습니다.) 오류가 발생해서 ```#include <chrono>```와 ```using namespace std::chrono_literals;```(chrono 네임스페이스에서 ms 같은 시간 단위 사용할 수 있게 해줌)를 추가했더니 오류가 해결 되었다. 

여기서 아쉬운 점은 스핀락처럼 계속 뺑뺑이를 돌며 lock이 풀렸는지 확인하는 점이다. 직접 확인하지 않고 누군가에게 확인을 부탁해보자.

---

### 이벤트
운영체제에서 제공하는 ```HANDLE```을 사용한다. ```HANDLE hEvent```는 특정 커널 오브젝트를 지칭하는 정수이다. 커널 오브젝트는 유저 레벨에서 관리하는 오브젝트와는 달리 운영체제가 관리하는 오브젝트를 생성할 수 있도록 요청하는 것이다.

```cpp
std::mutex m;
std::queue<int> q;
HANDLE hEvent;

void Producer()
{
	while (true)
	{
		std::unique_lock<std::mutex> lock(m);
		q.push(100);

		// 데이터를 넣은 후에 파란불 킴
		::SetEvent(hEvent);

		std::this_thread::sleep_for(100ms);
	}
}

void Consumer()
{
	while (true)
	{
		// 파란 불로 켜질 때까지 무한 대기 - 파란 불이 켜지면 깨어나서 실행할 수 있음
		::WaitForSingleObject(hEvent, INFINITE);

		std::unique_lock<std::mutex> lock(m);
		if (q.empty() == false)
		{
			int data = q.front();
			q.pop();
			std::cout << data << std::endl;
		}
	}
}

int main()
{
	// 커널 오브젝트
	// - Usage Count
	//   Signal (파란 불) / Non Signal (빨간 불)

	// Auto / Manual 둘 중 선택할 수 있음 -  bool
	hEvent = ::CreateEvent(NULL/*보안 속성*/, FALSE/*bManualReset*/, FALSE/*초기상태*/, NULL);

	std::thread t1(Producer);
	std::thread t2(Consumer);

	t1.join();
	t2.join();

	::CloseHandle(hEvent);
}
```

&nbsp;

이벤트 
- 장점 : 계속 대기타는 것보다 낭비를 줄일 수 있다.
- 단점 : 비용 문제 - 모든(짜잘한) 것에 대한 이벤트를 처리하는 것에 대해 문제가 있다.

### 조건 변수(condition variable)
- 조건 변수 : 커널 오브젝트가 아니라 유저 레벨 오브젝트이다. 흐름과 사용하는 방식은 이벤트와 비슷하다. 다른 점은 일이 있을 때만 실행할 수 있다는 것이다. 불필요한 낭비를 막을 수 있다!

```cpp
std::mutex m;
std::queue<int> q;
HANDLE hEvent;

// CV는 커널 오브젝트가 아니라 유저레벨 오브젝트
std::condition_variable cv;

void Producer()
{
	while (true)
	{
		// Condition Variable
		// 1) Lock을 잡기
		// 2) 공유 변수 값 수정
		// 3) Lock을 풀고
		// 4) CV 통해서 통지
		
		{
			std::unique_lock<std::mutex> lock(m);
			q.push(100);
		}

		cv.notify_one();
		std::this_thread::sleep_for(100ms);
	}
}

void Consumer()
{
	while (true)
	{
		//::WaitForSingleObject(hEvent, INFINITE);
		std::unique_lock<std::mutex> lock(m);
		cv.wait(lock, []() {return q.empty() == false; });
		// 1) Lock을 잡으려고 시도 (이미 잡혔으면 skip)
		// 2) 조건 확인
		// - 만족O => 바로 빠져 나와서 이어서 코드 진행
		// - 만족X => Lock을 풀어주고 대기 상태로 전환

		{
			int data = q.front();
			q.pop();
			std::cout << data << std::endl;
		}
	}
}

int main()
{
	// 커널 오브젝트
	// - Usage Count
	//   Signal (파란 불) / Non Signal (빨간 불)

	// Auto / Manual 둘 중 선택할 수 있음 -  bool
	hEvent = ::CreateEvent(NULL/*보안 속성*/, FALSE/*bManualReset*/, FALSE/*초기상태*/, NULL);

	std::thread t1(Producer);
	std::thread t2(Consumer);

	t1.join();
	t2.join();

	::CloseHandle(hEvent);
}
```

&nbsp;

기억할 것 : 스핀락처럼 무한이 확인하며 기다리지 않고 누군가 알려주는 방식이 있다. (나중에 필요할 때 구글링해서 찾으면 됨)

---

## 스마트 포인터(24.11.14)
생 포인터를 쓰게 되면 생명 주기가 꼬일 가능성이 매우매우 높다. 그래서 레퍼런스 카운트를 둬서 포인터를 관리하는 방법을 사용한다.

### 레퍼런스 카운트 수동 관리 실습
레퍼런스 카운트를 관리하는 방법
1. 객체에 포함 시킴.
2. 따로 레퍼런스 카운터를 관리하는 블록 생성함.

RefCountable class를 생성해서 수동으로 레퍼런스 카운트를 관리하도록 구현한다.
```cpp
class RefCountable
{
public:
	RefCountable() {}
	virtual ~RefCountable() {}

	int GetRefCount() { return _refCount; }
	int AddRef() { return ++_refCount; }
	int ReleaseRef()
	{
		int refCount = --_refCount;
		if (refCount == 0)
			delete this;
		
		return refCount;
	}

protected:
	int _refCount = 1;
};
```

&nbsp;

객체를 생성할 때 RefCountable을 상속받도록 한다.
```cpp
class Knight : public RefCountable
{
public:

};

std::map<int, Knight*> _knights;

int main() 
{
	Knight* knight = new Knight();	// _refCount == 1

	_knights[100] = knight;
	knight->AddRef();		// _refCount == 2;

	_knights.erase(100);
	knight->ReleaseRef();	// _refCount == 1
	knight = nullptr;
}
```

&nbsp;

멀티 스레드 환경이라면 일어날 수 있는 문제점 
- 증감 연산자가 어셈블리어로 3단계에 걸쳐 일어나기 때문에 refCount를 atomic으로 수정해야 한다. ```int _refCount = 1;``` -> ```std::atomic<int> _refCount = 1;```
- ```_knights[100] = knight;```와 ```knight->AddRef();``` 사이에 누군가 끼어들어서 Count 값을 0으로 만들어 메모리를 소멸시켜버릴 수 있다.
-> <span style="color:red"> 웬만하면 수동으로 레퍼런스 카운터 관리하지 말자. (atomic을 해도 소용이 없음) </span>

### 스마트 포인터
'복사가 되어 생성될 때 RefCount를 1 증가시킨다.'가 핵심! 위의 수동 방식과는 다르게 중간에 누가 끼어들어도 객체가 소멸하지 않는다.

주의할 점
- 순환 문제(나중에 더 자세히 다룰 예정)
- 스마트 포인터에서 this 포인터 문제(e.g. 이중 포인터 삭제 등)

정리
1. RefCount - 스마트 포인터를 이용해 간접적으로 몇 번 참조 되었는지 RefCount로 확인할 수 있다.
2. RefCount 수동 관리? - 중간에 누가 끼어들 수 있어 위험하다.
3. RefCount를 객체에 포함시킬 것인지(직접 구현할 때)?
4. SharedPtr은 왜 2번 문제를 해결해주나?
5. SharedPtr과 this의 문제를 어떻게 해결할 것인가?

---

출처 [루키스님 게임 프로그래밍 올인원 강의](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss)