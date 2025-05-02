---
title: "[DirectX 12] #4-1. 삼각함수"
excerpt: "삼각함수 개념과 게임에서의 활용"

categories:
  - DirectX12
tags:
  - [DirectX, TrigonometricFunction]

permalink: /DirectX12/trigonometric-function/

toc: true
toc_sticky: true

date: 2025-05-02
last_modified_at: 2025-05-02
---

## 들어가며

게임 개발에서 사용되는 수학은 생각보다 어렵지 않다. 대부분 삼각함수, 벡터, 행렬의 개념만 잘 익혀두면 충분하다. 이번에는 삼각함수에 대한 개념과 게임에서의 활용에 대해 알아보자.

---

## 피타고라스의 정리

직각 삼각형 ABC에서 

- 빗변: ℎ
- 밑변 (adjacent): 𝑎
- 높이 (opposite): 𝑜
- 각도: 𝜃

```math
h^2 = a^2 + o^2
```

&nbsp;

- 게임에서의 활용
    - 예: 플레이어와 몬스터 간의 거리 계산
        ```cpp
        float dx = monster.x - player.x;
        float dy = monster.y - player.y;
        float distance = sqrt(dx * dx + dy * dy); // 피타고라스 정리
        ```

---

## 삼각함수 

- 삼각함수의 기본 정의

```math
\cos(\theta) = \frac{a}{h}
```

```math
\sin(\theta) = \frac{o}{h}
```

```math
\tan(\theta) = \frac{o}{a} = \frac{\sin(\theta)}{\cos(\theta)}
```

---

### 단위원

- 정의
    - 반지름이 1인 원
    - 원 위의 각도 θ에 해당하는 점의 좌표
        ```math
        (\cos(\theta), \sin(\theta))
        ```

---

### 라디안

게임에서는 도(degree)가 아닌 라디안(radian)을 사용한다.

```math
180^\circ = \pi,\quad 360^\circ = 2\pi
```

```math
60^\circ = \frac{\pi}{3},\quad 45^\circ = \frac{\pi}{4},\quad 30^\circ = \frac{\pi}{6}
```

---

### 삼각함수의 성질

- 주기성
    ```math
    \sin(\theta + 2\pi) = \sin(\theta) \
    \cos(\theta + 2\pi) = \cos(\theta)
    ```
- 짝/홀 함수 (우/기 함수)
    ```math
    \cos(-\theta) = \cos(\theta) \quad \text{(짝함수)} \
    \sin(-\theta) = -\sin(\theta) \quad \text{(홀함수)}
    ```

---

### 역삼각함수

삼각함수의 값을 알고 있을 때 각도를 구하고 싶다면? (단 역함수는 결과의 범위가 정해져 있음)

```math
\theta = \cos^{-1}(x),\quad \theta \in [0, \pi] \
\theta = \sin^{-1}(x),\quad \theta \in \left[-\frac{\pi}{2}, \frac{\pi}{2} \right]
```

--- 

## 활용

- 거리 계산
    ```cpp
    float distance = sqrt(dx * dx + dy * dy);
    ```
- 두 벡터의 사이각 구하기
    ```cpp
    float dot = Normalize(a).Dot(Normalize(b));
    float angle = acos(dot);
    ```
- 코사인 덧셈 공식 
    - 회전 행렬에서 활용된다. 
        ```math
        \cos(\alpha + \beta) = \cos(\alpha)\cos(\beta) - \sin(\alpha)\sin(\beta)
        ```

---


---

## 마치며

오랜만에 삼각함수 개념을 공부했다. 바로 기억이 안나는 부분이 있어서 주기적으로 복습해야겠다!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*