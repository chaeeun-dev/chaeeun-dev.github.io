---
title: "[Server] #15-3. 쓰레드 생성"
excerpt: "C++로 쓰레드 생성하기 실습"

categories:
  - Server
tags:
  - [Server, Thread]

permalink: /theory/server/thread-creation-practice/

toc: true
toc_sticky: true

date: 2024-09-30
last_modified_at: 2024-09-30
---

## 메모리 영역과 쓰레드

멀티 쓰레드 환경에서 메모리 사용 방식을 살펴보자.
- Code 영역: 실행할 프로그램의 코드가 저장되는데, 수정이 되지 않기 때문에 쓰레드 간 충돌 문제가 없다.
- Stack 영역: 각 쓰레드는 고유한 스택을 가지고 있어 독립적으로 작동하기에 충돌 문제가 없다.
- Heap 영역: 동적으로 할당된 메모리로, 여러 쓰레드가 **공유**해서 사용할 수 있다.
- Data 영역: 전역 변수 및 정적 변수가 저장되며, 여러 쓰레드가 **공유**해서 사용할 수 있다. 

> Heap과 Data 영역은 여러 쓰레드가 공유하므로 경쟁 상태(Race Condition)가 발생할 위험이 있다.
> - 경쟁 상태(Race Condition)란?
>   - 멀티 쓰레드 프로그래에서 여러 쓰레드가 공유 자원에 동시에 접근하여 수정하는 상황에서 발생하는 문제이다.
>   - 여러 쓰레드가 공유 자원을 동시에 접근하면 오류가 발생할 수 있기 때문에 이를 방지하기 위해 `Mutex`, `Atomic` 변수, `Lock` 등의 동기화 기법이 필요하다.

---

## 쓰레드 생성 실습

### 헤더 파일

- `thread`와 스레드를 담을 `vector` 헤더 파일을 include한다.

```cpp
#include <iostream>
using namespace std;
#include <thread>
#include <vector>
```

---

### HelloThread 함수

- 각 쓰레드가 실행될 때 호출되는 함수이다.
- 쓰레드 번호를 출력하는 무한 루프를 실행한다.

```cpp
void HelloThread(int i)
{
    while (true)
    {
        cout << "Hello Thread!" << i << endl;
    }
}
```

---

### main 함수

- 쓰레드 생성
    - `vector<thread>`를 사용해 12개의 쓰레드를 생성하고 저장한다. 
    - 각 쓰레드는 `HelloThread(i)` 함수를 실행한다.

```cpp
vector<thread> threads;

for (int i = 0; i < 12; ++i)
{
    threads.push_back(thread(HelloThread, i));
}
```

&nbsp;

- 쓰레드 종료 처리
    - `hardware_concurrency()` 함수는 CPU 코어 개수를 반환한다.
        - CPU 코어보다 많은 쓰레드를 생성한다고 해서 프로그램이 더 빨라지지 않는다.
        - 보통 코어 개수만큼의 쓰레드가 적절한 성능을 보장한다. 
    - `t.join()`을 호출하여 메인 쓰레드가 모든 자식 쓰레드의 종료를 기다리도록 설정한다. `join()`을 하지 않으면 메인 쓰레드가 종료되면서 프로그램이 비정상적으로 종료될 수 있다.

```cpp
for (thread& t :threads)
{
    int a = t.hardware_concurrency();

    t.join();
}
```

---

[실습 커밋](https://github.com/chaeeun-dev/Server/commit/5515179eb2fb9734efdffb1361c9d4f9cbeec439#diff-5461936725ce266bb330a42086e4adb296e17e78e88f466d776579e341ed8620)

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*