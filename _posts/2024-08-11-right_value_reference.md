---
title: "[C++] r-value, move constructor"
date: 2024-08-11 12:50:00 +09:00 
categories: C++
author_profile: false
toc : true
toc_sticky: true
---


## 왼 값(l-value)과 오른 값(r-value)

- 왼 값 : 메모리 주소를 가지고 있는, 메모리 상에 위치가 있는 값
- 오른 값 : 임시 값, 메모리에 직접 위치하지 않는 값. 오른 값은 보통 임시 객체이며 곧 사라질 값을 의미함

int x = 10; 에서 x는 왼 값, 10은 오른 값

## 왼 값 참조(l-value reference)

왼 값 참조는 T& 형태로 선언됨

```cpp
void TestKnight_LValueRef(Knight& knight)
{
			knight.hp = 100;  // 원본에 영향을 줌
}
```


## 오른 값 참조(r-value reference)

오른 값  참조는 T&& 형태로 선언됨, 주로 이동이나 소유권 이전 할 때 사용됨

```cpp
void TestKnight_RValeRef(Knight&& knight)
{
		// knight는 더 이상 사용되지 않을 것이라는 가정하에 작업 진행
}
```


## 이동 생성자와 이동 대입 연산자

- 자원의 이동을 위해 사용됨
- 이동 생성자는 객체가 새롭게 생성될 때 호출, 이동 대입 연산자는 이미 존재하는 객체에 다른 객체의 자원을 이동할 때 호출
- 이동 생성자와 이동 대입 연산자는 일반적으로 오른 값 참조를 사용하여 구현됨

```cpp
Knight(Knight&& knight) noexcept
{
    // 이동 생성자: 자원의 소유권을 이동
    _hp = knight._hp;
    _pet = knight._pet;
    knight._pet = nullptr;  // 원본 객체는 더 이상 소유하지 않음
}

void operator=(Knight&& knight) noexcept
{
    // 이동 대입 연산자: 자원의 소유권을 이동
    _hp = knight._hp;
    _pet = knight._pet;
    knight._pet = nullptr;  // 원본 객체는 더 이상 소유하지 않음
}
```

std::move  - 객체를 오른 값으로 반환해주는 함수, 이를 통해 객체의 자원을 안전하게 이동할 수 있음

k2 = std::move(k1);   k1을 오른 값으로 변환하여 k2에 이동 대입 연산을 수행


cf) 복사 생성자와 복사 대입 연산자
```cpp
Knight(const Knight& knight)
{
			// 복사 생성자 : 기본적인 얕은 복사
}

void operator=(const Knight& knight)
{
			// 복사 대입 연산자 : 깊은 복사를 구현
}
```


## 정리

- 왼 값 참조 : 메모리에 존재하는 변수를 참조, 일반적으로 변수를 대상으로 사용
- 오른 값 참조 : 임시 객체나 더 이상 사용되지 않을 객체의 자원을 효율적으로 이동할 때 사용
- 이동 생성자와 이동 대입 연산자 : 객체의 자원 소유권을 이동, 낭비를 줄임