---
title: "[Server] #15-9. 이벤트와 조건 변수"
excerpt: "Producer & Consumer 패턴으로 이벤트와 조건 변수 활용하기"

categories:
  - Server
tags:
  - [Server, Event, ConditionVariable]

permalink: /theory/server/event-and-condition-variable/

toc: true
toc_sticky: true

date: 2024-11-13
last_modified_at: 2025-03-14
---

## 이벤트와 조건 변수

멀티 스레드 환경에서 데이터를 안전하게 공유하면서 대기 시간을 줄이는 방법을 알아본다. 이번에는 Producer & Consumer 패턴을 사용하여 이벤트(Event)와 조건 변수(Condition Variable)을 활용하는 방법을 알아보자.

---

### Producer & Consumer

생산자와 소비자로 역할을 분담하면, 여러 스레드가 동시에 데이터를 처리하면서도 락 경합을 줄일 수 있다.

&nbsp;

기본적인 Producer & Consumer 패턴을 코드로 구현해보자.

- `Producer()`
  - 무한 루프를 돌며 큐에 데이터를 넣는다.
  - `std::unique_lock<std::mutex>`를 사용해 뮤텍스를 잠그고, 큐에 데이터(`100`)를 넣은 후 잠금을 해제한다.
  - 데이터를 넣은 후 100ms 동안 잠든다.
- `Consumer()`
  - 무한 루프를 돌며 큐에서 데이터를 꺼낸다.
  - `std::unique_lock<std::mutex>`를 사용해 뮤텍스를 잠그고, 큐가 비어있지 않으면 데이터를 꺼내서 출력한다.
  - 큐가 비어있으면 아무런 작업도 하지 않는다.

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

- 문제점
  - Busy Waiting
    - 소비자 스레드는 큐가 비어있을 때도 무한 루프를 돌면서 큐를 확인한다. 아는 CPU 자원을 낭비하는 Busy Waiting 문제다.
  - 소비자 스레드가 직접 확인하지 않고, 누군가 알려주는 방식(큐에 데이터가 추가될 때까지 소비자 스레드는 대기 상태가 될 수 있음)으로 개선할 수 있다. → 이벤트 or 조건 변수를 활용


> 💡 Busy Waiting이란?
>
> 스레드나 프로세스가 특정 조건이 충족(e.g. 락이 해제, 데이터 도착 등)될 때까지 무한 루프를 돌면서 조건을 확인하는 방식이다.
>
> 스레드가 대기 상태로 전환되지 않고 계속 실행 상태를 유지하므로, CPU 자원을 불필요하게 소모한다.

---

### Event

운영체제에서 제공하는 커널 오브젝트(HANDLE) 를 활용하면, 락이 풀릴 때까지 직접 확인하지 않고운영체제가 신호(Signal)를 보내도록 할 수 있다.

> `HANDLE hEvent`는 특정 커널 오브젝트를 지칭하는 정수이다.
>
> 커널 오브젝트는 유저 레벨에서 관리하는 오브젝트와 달리, 운영체제가 관리하는 오브젝트를 생성할 수 있도록 요청한다.

&nbsp; 

이벤트를 사용한 코드를 살펴보자.

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

		// 데이터를 넣은 후 이벤트 신호 설정
		::SetEvent(hEvent);

		std::this_thread::sleep_for(100ms);
	}
}

void Consumer()
{
	while (true)
	{
		// 이벤트 신호가 올 때까지 대기 (무한 대기)
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
	// 커널 오브젝트 생성 (이벤트 초기화)
	hEvent = ::CreateEvent(NULL, FALSE, FALSE, NULL);

	std::thread t1(Producer);
	std::thread t2(Consumer);

	t1.join();
	t2.join();

	::CloseHandle(hEvent);
}
```

- 이벤트의 장단점
  - 장점
    - Busy Waiting 문제 해결 - 스레드가 직접 락을 확인하지 않고, 이벤트 신호를 기다리기 때문에 CPU 자원을 효율적으로 사용할 수 있다.
    - 커널 오브젝트이기 때문에 프로세스 간 동기화에도 사용할 수 있다.
  - 단점
    - 운영체제가 관리하는 커널 오브젝트이므로 비용이 발생한다.
    - 이벤트를 과도하게 사용하면 커널 오브젝트 관리 비용이 증가해 성능이 저하될 수 있다.

---

### Condition Variable

조건 변수는 운영체제가 아닌 유저 레벨에서 관리하는 오브젝트이다. 이벤트와 사용 방식은 비슷하지만, 커널 오브젝트를 사용하지 않아 성능 부담이 적다.

&nbsp;

조건 변수를 사용한 코드를 살펴보자. 

```cpp
std::mutex m;
std::queue<int> q;
std::condition_variable cv;

void Producer()
{
	while (true)
	{
		{
			std::unique_lock<std::mutex> lock(m);
			q.push(100);
		}

		// 조건 변수 알림
		cv.notify_one();
		std::this_thread::sleep_for(100ms);
	}
}

void Consumer()
{
	while (true)
	{
		std::unique_lock<std::mutex> lock(m);
		
		// 조건을 만족할 때까지 대기 (락을 자동으로 풀었다가 깨어날 때 다시 잡음)
		cv.wait(lock, []() { return q.empty() == false; });

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

- 조건 변수의 장단점
  - 장점
    - Busy Waiting 문제 해결 - 조건 변수는 스레드가 조건을 만족할 때까지 대기 상태로 들어가기 때문에 CPU 자원을 효율적으로 사용할 수 있다.
    - 이벤트와 유사하지만, 운영체제 개입 없이 유저 레벨에서 동작한다. (커널 모드 전환 비용이 없음)
    - 스레드가 직접 lock을 확인하지 않아 busy waiting 문제를 해결한다.
    - 운영체제에서 관리하는 커널 오브젝트를 사용하지 않기 때문에 비용이 절감된다.
  - 단점
    - 커널 오브젝트가 필요한 상황(e.g. 프로세스 간 동기화)에서는 조건 변수로 대체할 수 없다.
    - 특정 상황에서는 커널 오브젝트가 더 적합할 수 있다.

---

## 마치며

- 기억할 것 
  - 스핀락처럼 계속 확인하며 기다리지 않고 "누군가 알려주는 방식"을 사용하면 CPU 자원을 아낄 수 있다.
  - 나중에 필요할 때 구글링 해서 구현할 수만 있으면 된다.

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*