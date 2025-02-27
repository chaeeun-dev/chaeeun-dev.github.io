---
title: "[Server] #15-5. 공유 자원"
excerpt: "공유 자원 실습을 통해 문제와 원인을 분석하고 해결하기"

categories:
  - Server
tags:
  - [Server]

permalink: /theory/server/shared-resource/

toc: true
toc_sticky: true

date: 2024-10-07
last_modified_at: 2025-02-27
---

## 공유 자원

멀티 쓰레드 환경에서는 여러 쓰레드가 동시에 같은 변수를 수정하면 데이터 충돌(Race Condition)이 발생할 수 있다.

---

### 실습1 - 공유 자원 문제 확인하기

- 공유 자원의 문제를 디스어셈블리로 확인해본다.

```cpp
int sum = 0;	// Data 영역

void Add()
{
	for (int i = 0; i < 100'0000; i++)
	{
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
}
```

- 결과
    - `i`가 100일 때 `sum == 0`
    - `i`가 100'0000일 때 `sum == 178827`
    - 이론적으로 0이어야 할 `sum`값이 0이 아닐 수도 있다. (예측 불가능한 값)
- 원인
    - `sum++`, `sum--` 연산이 세 단계(load, modify, store)로 이루어져 있는데, 이 과정에서 다른 스레드가 끼어들면 값이 덮어씌워져서 연산이 손실될 수 있다. → 경쟁 조건(Race Condition)이 발생
    - 디스어셈블리 창으로 계산을 확인하면, `sum++`은 `int eax = sum;` → `eax = eax + 1;` → `sum = eax;` 이렇게 세 단계에 걸쳐 실행된다.
    - 계산이 한 번에 이뤄지지 않는 이유는, Data 영역에 있는 `sum` 변수는 RAM만 알고 CPU가 알지 못하기 때문에 레지스터에 넣은 값으로 연산을 수행한다.

---

### 실습2 - atomic으로 문제 해결하기

`std::atomic`을 사용하면 모든 연산을 단일 명령어로 처리하여 데이터 경합을 방지한다.

```cpp
std::atomic<int> sum = 0;

void Add()
{
	for (int i = 0; i < 100'0000; i++)
	{
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
}
```

- 결과
    - `sum == 0`이 보장된다.
- 원리
    - `sum++`이 기존의 `inc` 명령어에서 `lock inc` 명령어로 변경되어 원자적 연산이 수행된다.

> `atomic`은 CPU의 lock prefix(예. `lock inc`)를 사용하여 명령어를 단일 연산으로 만들어준다.

---

### 실습3 - atomic 함수 

```cpp
	std::atomic<int> sum = 0;

	int temp = sum.load();	
	
	sum.store(10);	

	int temp = sum.exchange(10);	
```

- `load()`: 값 읽기 (일반 변수 읽기와 동일하지만 원자적 연산)
- `store()`: 값 저장하기 (일반 변수 저장과 동일하지만 원자적 연산)
- `exchange()`: 값을 변경하면서 이전 값 반환하기

---

### 실습4 - Stack, Heap에서의 공유 자원

Heap에서 공유 자원을 만들어보고 Crash가 발생하는지 확인해본다.

```cpp
void Test()
{
	for (int i = 0; i < 10000; i++)
	{
		int* p = new int();		

		*p = 100;

		delete p;
	}
}
```

- 결과
    - Crash가 발생하지 않는다.
    - Heap을 사용해도 자원이 공유되지 않으면 문제가 발생하지 않는다.
    - 스레드 간 자원이 공유될 경우 문제가 발생할 수 있다.
        - 예. 전역 변수, 함수 인자로 포인터 공유
    
![image](https://github.com/user-attachments/assets/b133a9dc-52e9-474d-847e-7927f5c9868a)

- 포인터 `p`
    - Stack 영역에 저장된다.
    - `p` 자체는 힙 메모리를 가리키는 주소를 보관한다.
- `new int()`로 할당된 메모리
    - Heap 영역에 저장된다.
    `*p = 100`을 실행하면, 힙 메모리에 있는 공간에 100을 저장한다. 
- 정리
    - 스택: 포인터 변수 `p`가 저장된다. (주소만 보관)
    - 힙: `new int()`로 할당된 메모리 공간이 저장된다. (실제 값 저장)
    - `delete p;`를 호출하지 않으면, 힙 메모리에 할당된 공간이 해제되지 않는다.
    - → `p`는 스택 영역에 있기 때문에 다른 스레드들이 접근할 수 없다.
--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*