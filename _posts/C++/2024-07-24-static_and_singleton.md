---
title: "[C++] static과 singleton"
date: 2024-07-24 23:20:00 +09:00 
categories: C++
author_profile: false
toc : true
toc_sticky: true
---

### static과 singleton

## static

정의) 클래스의 모든 instance가 공유하는 변수나 함수

특징)

- 클래스의 모든 객체가 동일한 static 변수를 사용한다
- 사용 예) Player instance들의 ID를 생성할 때, Archer 의 모든 instance의  attack 속성이 10으로 같아야 할 때 등
- static 멤버 변수는 클래스의 외부에서 정의해야 함

```jsx
class Player()
{
public: 
	static int mp;
	static void f() { }
};

int Player::mp = 10;
```

## singleton (디자인 패턴) ← 아직 자세하게 다루지 않았음!!

정의) 클래스의 instance가 하나만 존재하도록 보장하는 디자인 패턴
특징)

- 특정 클래스의 instance가 하나만 생성되고, 전역적으로 접근할 수 있음.

차이점

- static은 클래스에서 함수나 변수를 공유할 때 사용
- singleton은 클래스의 instance를 하나로 제한할 때 사용
