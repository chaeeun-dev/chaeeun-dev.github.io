---
title: "[Server] #15-4. 캐시와 CPU 파이프라인"
excerpt: "캐시와 CPU 파이프라인에 대한 실습과 최적화 위험성 알아보기"

categories:
  - Server
tags:
  - [Server]

permalink: /theory/server/cache-and-cpu-pipeline/

toc: true
toc_sticky: true

date: 2024-10-03
last_modified_at: 2025-02-27
---

## 캐시(Cache)

CPU는 연산을 담당하고, RAM은 데이터를 저장하는 역할을 한다. 그런데 CPU와 RAM 사이의 거리가 멀어 데이터를 직접 주고받는 것은 비효율적이다. 이를 해결하기 위해 **캐시(Cache)**가 사용된다.

- 캐시의 원리 (캐시 철학)
    1. 시간 지역성(Temporal Locality)
        - 최근에 접근한 데이터는 다시 접근할 확률이 높다.
        - 예. 같은 변수를 여러 번 사용할 때
    2. 공간 지역성(Spatial Locality)
        - 특정 메모리 위치에 접근하면, 그 근처의 메모리도 접근할 확률이 높다.
        - 예. 배열을 순차적으로 탐색하 때

---

### 실습1 - 캐시 성능 비교

- 캐시 히트(Cache Hit)가 성능에 미치는 영향을 실험해본다.

```cpp
int buffer[10000][10000];

int main()
{
	std::memset(buffer, 0, sizeof(buffer));	// 0으로 밀어 넣음

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
				sum += buffer[j][i];    // j, i 순서만 바꿈

		auto end = GetTickCount64();

		std::cout << "Elapsed Tick" << (end - start) << std::endl;

	}
}
```

- 결과 
    - `Elapsed Tick 94, Elapsed Tick 375`
    - 첫 번째 루프는 순차적으로 메모리를 읽어 **캐시 히트(Cache Hit)**가 자주 발생하여 속도가 빠르다.
    - 두 번째 루프는 불연속적으로 메모리르 읽어 **캐시 미스(Cache Miss)**가 많이 발생하여 속도가 느려진다.

> 💡 결론: 메모리를 접근할 때는 순차적으로 접근하는 것이 캐시 활용을 극대화하는 방법이다.

---

## CPU 파이프라인

CPU는 한 번에 한 명령어씩 처리하는 게 아니라, 여러 단계를 겹처서 관리할 수 있다.

- 4단계 CPU 파이프라인
    1. Fetch: 명령어를 메모리에서 가져온다.
    2. Decode: 명령어를 해석한다.
    3. Execute: 명령어를 실행한다.
    4. Write-back: 실행 결과를 저장한다.
- 비유 - 세탁기, 건조기, 다리미
    - 세탁물을 하나씩 세탁 → 건조 → 다림질하면 오래걸린다.
    - 세탁기가 동작하는 동안 건조기를 동시에 사용하고, 그동안 다림질하면 훨씬 효율적이다.
    - CPU도 마찬가지로 한 작업이 끝나기를 기다리지 않고, 여러 단계가 겹쳐서 실행된다.

![image](https://github.com/user-attachments/assets/abfdcdb6-6141-4e96-816d-0eeb5a0b4bde)

---

### 실습2 - 파이프라인 최적화의 위험성

- CPU가 명령어 실행 순서를 최적화하면서 의도치 않은 결과가 발생할 수 있음을 확인해본다.

```cpp
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

![Image](https://github.com/user-attachments/assets/f4137b52-ed19-4858-90e0-6c127b7db8c6)

- 결과
    - `r1 == 0 && r2 == 0`이 발생하는 경우, CPU가 명령어 순서를 최적화하여 의도한 실행 순서가 어긋났음을 의미한다. (코드의 의도대로라면, `r1 == 0 && r2 == 1`이어야 하므로, while문이 종료되면 안 됨)
    - 멀티 쓰레드 환경에서는 명령어 실행 순서가 예측과 다를 수 있어 버그가 발생할 가능성이 크다.

> 💡 결론: 멀티 쓰레드 환경에서는 CPU 최적화로 인해 의도치 않은 결과가 나올 수 있어, 순서가 중요한 연산에 대해 동기화 처리가 필수적이다.

---

### 실습3 - 컴파일러 최적화 방지

컴파일러는 코드 실행을 최적화하기 위해 불필요한 연산을 생략할 수 있다. 하지만 멀티 쓰레드 환경에서는 최적화가 문제를 일으킬 수 있다.

- 최적화를 방지하는 코드를 살펴보자.

```cpp
volatile bool ready = false;

void Thread_1()
{
	while (ready == false){ }	
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

- volatile 키워드
    - 컴파일러가 변수를 최적화하지 않도록 강제할 수 있다.
    - `while (ready == false)`가 최적화되어 무한 루프가 제거되는 것을 방지한다.

> 💡 결론: 멀티 쓰레드 환경에서는 최적화로 인해 예상치 못한 버그가 발생할 수 있으며, 이를 방지하기 위해 `volatile`, `atomic` 등의 기법을 사용해야 한다. 

---

### 컴파일러 최적화 vs CPU 최적화

- 컴파일러 최적화
    - 컴파일러가 코드를 분석하여 실행 속도를 높이거나 코드 크기를 줄이는 작업이다.
    - 예를 들어
        - 불필요한 연산 제거 (`while (ready == false)` → `while (false)`)
        - 명령어 재배치
        - 변수 제거 (사용되지 않는 변수 삭제)
- CPU 최적화
    - CPU 내부에서 실행되는 명령어를 최적화하여 성능을 높이는 작업이다.
    - 예를 들어
        - 파이프라인 명령어 재정렬 → 실행 순서를 CPU가 알아서 조정하여 빠르게 실행한다.
        - 분기 예측 → 반복문이 실행될지를 미리 예측하여 불필요한 연산을 최소화한다.
        - 캐시 활용 → 자주 쓰는 데이터를 캐시에 저장하여 메모리 접근 속도를 향상시킨다.

&nbsp;

- `volatile` 키워드는 컴파일러의 최적화를 막지만, CPU의 최적화는 막을 수 없다.
    - `volatile`이 막는 것
        - 컴파일러가 변수를 캐싱하거나 불필요한 연산을 제거하는 것을 방지한다.

        ```cpp
        volatile bool ready = false;

        void Thread_1()
        {
            while (ready == false) { }  // 컴파일러가 무한 루프 제거하지 않음
            std::cout << "Thread 실행!" << std::endl;
        }
        ```

        - 컴파일러가 `ready`값을 캐싱하지 않고, 항상 메모리에서 읽어오도록 강제한다.
        - 만약 `volatile`이 없으면, 컴파일러가 `ready == false`를 항상 거짓으로 가정하여 루프를 제거할 수 있다.
    - `volatile`이 막지 못하는 것
        - CPU가 명령어를 재배열하는 것은 막지 못한다.

        ```cpp
        volatile int x = 0;
        volatile int y = 0;

        void Thread_1() {x = 1; y = 2;}
        ```

        - 실행 순서 `x = 1` → `y = 2`가 보장되지 않는다. 
        - CPU 내부 최적화를 방지하려면, `std::atomic`을 사용해야 한다.
        - 참고. `atomic`은 컴파일러와 CPU의 최적화를 둘 다 방지한다.

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*