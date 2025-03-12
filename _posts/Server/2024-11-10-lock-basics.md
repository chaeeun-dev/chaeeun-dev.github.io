---
title: "[Server] #15-6. Lock 기초"
excerpt: "Mutex와 RAII 개념을 이해하고 LockGuard 사용하기"

categories:
  - Server
tags:
  - [Server, Lock, LockGuard, Mutex, RAII]

permalink: /theory/server/lock-basics/

toc: true
toc_sticky: true

date: 2024-11-10
last_modified_at: 2025-03-12
---

## Mutex

C++에서 mutex는 **mutal exclusion(상호 배제)**를 의미하며, 한 번에 하나의 스레드만 실행되도록 보장하는 역할을 한다.

- 사용 방법

```cpp
#include <mutex>

std::mutex m;
```

---

### 실습1 - Mutex가 없으면 발생하는 문제

다음 코드의 결과를 예측해보자.
1. 20000
2. Crash 발생
3. 이상한 값

```cpp
#include <vector>
#include <thread>
#include <iostream>

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

- 결과
    1. Crash가 발생한다. 
    2. 이상한 값이 나온다. (`vector.reverse(1000000)`을 사용해 이사 횟수를 줄였을 때)
- 원인
    1. `push_back()`을 호출할 때 데이터가 가득 차면 이사(rellocation)가 발생하는데, 한 스레드가 vector의 메모리를 재할당하면서 기존의 메모리를 해제하는 순간 다른 스레드가 해제된 메모리에 접근하면 크래시가 발생한다.
    2. `vector`는 동적 배열로 Heap 영역에 할당되므로 여러 스레드가 동시에 `push_back()`을 실행하며 같은 메모리 위치를 덮어쓴다.
- 해결 방법
    - `mutex`를 사용하여 `push_back()`이 동시에 실행되지 않도록 보호한다.

---

### 실습2 - Mutex를 이용한 문제 해결

한 번에 하나의 스레드만 실행할 수 있도록 `lock`으로 잠금을 걸고, 작업이 끝나면 `unlock`으로 잠금을 풀어준다.

```cpp
#include <vector>
#include <thread>
#include <iostream>
#include <mutex>

std::vector<int> v;
std::mutex m;

void Push()
{
    for (int i = 0; i < 10000; i++)
    {
        m.lock();
        v.push_back(i);
        m.unlock();
    }
}

int main()
{
    std::thread t1(Push);
    std::thread t2(Push);

    t1.join();
    t2.join();

    std::cout << v.size() << std::endl;  // 정상적으로 20000 출력
}
```

- 결과
    - `v.size() == 20000`이 정상적으로 출력된다.
    - `push_back()`이 한 번에 하나의 스레드에서만 실행되도록 보장된다.
- Mutex 동작 방식
    - `m.lock()`: 다른 스레드가 접근하지 못하도록 잠근다.
    - `m.unlock()`: 작업이 끝난 후 잠금을 해제한다.
    - lock을 제대로 해제하지 않으면 프로그램이 멈추는 데드락(무한 대기)이 발생할 수 있다. 
    - 참고. 스레드가 실행되는 순서는 알 수 없다.

---

### 실습3 - RAII 패턴

- RAII(Resource Acquisition Is Initialization) 패턴을 사용하면 `lock()`과 `unlock()`을 직접 호출할 필요 없이 자동으로 관리할 수 있다.
- 자원을 수동으로 할당하고 해제하면(`lock` & `unlock`) 예외 발생시 자원 해제를 깜빡해서 mutex가 영구적으로 잠길 수 있다. RAII 패턴을 적용하면 더 안전하게 관리할 수 있다.

> 💡 RAII(Resource Acquisition Is Initialization)이란?
>
> 객체의 생명 주기와 자원의 관리를 연관시켜, 자원의 누수를 방지하는 C++ 프로그래밍 기법이다. 
> - 객체가 생성될 때 자원을 획득하고, 객체가 소멸될 때 자원을 자동으로 반환하도록 설계한다.
> - C++의 스코프 기반 자동 해제(자동 정리) 개념을 활용한다.
> - `std::lock_guard`, `std::unique_lock`, `std::shared_ptr` 등의 표준 라이브러리도 RAII 원칙을 따른다.

&nbsp;

- 직접 구현하기
    - 템플릿을 사용해 `lockGuard` 객체가 생성될 때 `lock()`을 호출하고, 소멸될 때 `unlock()`을 자동으로 호출한다.

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
        v.push_back(i);
    }
}
```

&nbsp;

- 표준 라이브러리 활용하기
    - C++ 표준에서 `std::lock_guard`를 제공하므로 `LockGuard`를 직접 만들지 않아도 된다.
    - 자동으로 `lock()`과 `unlock()`을 관리해주므로 데드락을 방지할 수 있다.
```cpp
void Push()
{
    for (int i = 0; i < 10000; i++)
    {
        std::lock_guard<std::mutex> lockGuard(m);
        v.push_back(i);
    }
}
```

--- 

### 실습4 - unique_lock

`std::lock_guard`와 유사하지만, `lock()`을 나중에 호출하거나, 조건에 따라 해제하는 기능이 필요할 때 사용한다. 

```cpp
void Push()
{
    for (int i = 0; i < 10000; i++)
    {
        std::unique_lock<std::mutex> lock(m);
        v.push_back(i);
        lock.unlock();  // 중간에 잠금을 해제할 수도 있음
    }
}
```

---

## 마치며

- 멀티스레드 환경에서 lock없이 `vector` 같은 STL 컨테이너에 접근하는 코드를 보면 바로 문제를 알아챌 수 있어야 한다. 
- lock을 남발하면 오히려 성능이 나빠질 수 있으니 적절히 사용해야 한다.

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*