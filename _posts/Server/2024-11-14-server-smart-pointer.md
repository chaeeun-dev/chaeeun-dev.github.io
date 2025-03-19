---
title: "[Server] #15-10. 스마트 포인터"
excerpt: ""

categories:
  - Server
tags:
  - [Server]

permalink: /theory/server/smart-pointer/

toc: true
toc_sticky: true

date: 2024-11-14
last_modified_at: 2025-03-19
---

## 들어가며

생 포인터(raw pointer)를 직접 사용하면 객체의 생명 주기 관리가 어려워지고, 메모리 이슈가 발생할 확률이 높다. 이를 해결하기 위해 레퍼런스 카운트를 사용하여 포인터를 관리하는 방식을 사용한다. 이번 수업에서는 레퍼런스 카운트를 직접 관리하는 실습과 스마트 포인터 실습을 통해 메모리를 관리하는 방법을 배운다.

---

## 레퍼런스 카운트 수동 관리

레퍼런스 카운트(Reference Count) 방식은 객체가 몇 개의 포인터에 의해 참조되고 있는지 추적하여, 더 이상 참조되지 않으면 자동으로 객체를 삭제하는 방법이다.

레퍼런스 카운트를 관리하는 방법은 크게 두 가지가 있다.
1. 객체 내부에 포함하여 관리하는 방식
2. 별도의 레퍼런스 카운터 블록을 생성하는 방식

&nbsp;

`RefCountable` 클래스를 통해 수동으로 레퍼런스 카운트를 관리해본다.

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

- `refCount` 값을 증가/감소시켜 현재 참조 개수를 추적한다.
- 참조 개수가 0이 되면 자동으로 `delete`가 수행된다.

&nbsp;

이제 `RefCountable`을 상속받아 객체를 관리해본다.

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

- 멀티 스레드 환경에서 발생할 수 있는 문제
    - `refCount` 증가/감소 연산은 3단계(읽기 → 수정 → 저장)로 이루어지므로, **여러 스레드가 동시에 접근할 경우 값이 꼬일 수 있다.** ([#15-5. 공유 자원](https://chaeeun-dev.github.io/theory/server/shared-resource/#%EC%8B%A4%EC%8A%B52---atomic%EC%9C%BC%EB%A1%9C-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0) atomic 내용 참고) → `std::atomic<int> _refCount = 1;`로 변경 필요
    - `Knight→AddRef();` 호출 전에 다른 스레드가 `ReleaseRef()`를 호출하여 객체가 삭제될 위험이 있다.
- 결론
    - **수동으로 레퍼런스 카운트를 관리하지 말자.** 
    - `std::atomic`을 사용해도 교착 상태(DeadLock)나 데이터 경합(Race Condition) 문제를 완전히 해결할 수 없다.

---

## 스마트 포인터

스마트 포인터는 **객체가 복사될 때 RefCount를 자동으로 증가**시켜 중간에 다른 스레드가 개입해도 객체가 소멸되지 않도록 보장하는 기능을 제공한다. 

&nbsp;

스마트 포인터 주의할 점
- 순환 참조 문제 발생 가능 (`shared_ptr`이 서로를 참조하면 참조 카운트가 0이 되지 않아 객체가 삭제되지 않는 문제) → `weak_ptr`로 해결
- `shared_ptr`에서 this 포인터 사용 문제 발생 가능 (클래스 내부에서 `shared_ptr`을 직접 삭제하면 이중 해제 문제가 발생할 수 있음)

&nbsp;

> 💡`weak_ptr`이 순환 참조를 방지하는 방법
>
> - `shared_ptr`이 복사될 대 컨트롤 블록의 RefCount가 증가하고, 마지막 `shared_ptr`이 소멸되면 RefCount가 0이 되어 객체도 자동 삭제된다.
> - `weak_ptr`은 RefCount를 증가시키지 않고 컨트롤 블록을 유지하여 순환 참조 문제를 방지한다.

&nbsp;

> 💡 컨트롤 블럭이란?
>
> 컨트롤 블록에는 객체를 안전하게 관리하기 위한 정보가 포함돼 있다. `shared_ptr`이 객체를 생성할 때, 실제 객체 외에 RefCount를 포함한 컨트롤 블록을 생성한다. 
> 1. RefCount: `shared_ptr`이 몇 개 존재하는지 추적
> 2. 객체 포인터: 실제로 관리하는 객체
> 3. WeakCount: `weak_ptr`이 몇 개 존재하는지 추적 (순환 참조 문제 해결)

---

## 마치며

- 스마트 포인터를 사용하면 수동 메모리 관리보다 안전하고 편리하다.
- 순환 참조 문제와 this 포인터 문제를 주의하자.

--- 

*출처*
*[Rookiss님 게임 프로그래머 입문 올인원](https://www.inflearn.com/course/%EA%B2%8C%EC%9E%84-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8-%EC%9E%85%EB%AC%B8-%EC%98%AC%EC%9D%B8%EC%9B%90-rookiss/dashboard)*