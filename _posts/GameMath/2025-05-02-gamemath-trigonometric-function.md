---
title: "삼각함수"
excerpt: "삼각함수 개념과 게임에서의 활용"

categories:
  - GameMath
tags:
  - [GameMath, TrigonometricFunction]

permalink: /GameMath/trigonometric-function/

toc: true
toc_sticky: true

date: 2025-05-02
last_modified_at: 2025-05-07
---

## 들어가며

게임 개발에서 사용되는 수학은 생각보다 어렵지 않다. 대부분 삼각함수, 벡터, 행렬의 개념만 잘 익혀두면 충분하다. 이번에는 삼각함수에 대한 개념과 게임에서의 활용에 대해 알아보자.

---

## 삼각함수

삼각함수에 대해 알아보자.

---

### 피타고라스의 정리

직각 삼각형 ABC에서 

- 빗변 (hypo-tenuse): ℎ
- 밑변 (adjacent): 𝑎
- 높이 (opposite): 𝑜
- 각도: 𝜃

![피타고라스의 정리](/assets/images/post_img/gamemath/PythagorasTheorem.jpg)

&nbsp;

- 게임에서의 활용
    - 예: 플레이어와 몬스터 간의 거리 계산
        ```cpp
        float dx = monster.x - player.x;
        float dy = monster.y - player.y;
        float distance = sqrt(dx * dx + dy * dy); // 피타고라스 정리
        ```

---

### 삼각함수 

- 삼각함수의 기본 정의

![삼각함수](/assets/images/post_img/gamemath/TrigonometricFunction.jpg)

---

### 단위원

- 정의
    - 원점을 중심으로 하고 반지름이 1인 원
    - 원 위의 각도 θ에 해당하는 점의 좌표

![UnitCircle](/assets/images/post_img/gamemath/UnitCircle.jpg)

---

### 라디안

게임에서는 도(degree)가 아닌 라디안(radian)을 사용한다.

- 라디안
    - 호의 길이가 반지름과 같게 되는 만큼의 각을 1 라디안이라고 정의한다. 

![Radian](/assets/images/post_img/gamemath/Radian.jpg)

---

### 삼각함수 그래프

![Graph](/assets/images/post_img/gamemath/TrigonometricFunctionGraph.jpg)


---

### 역삼각함수

삼각함수의 값을 알고 있을 때 각도를 구하고 싶다면 역삼각함수를 사용한다. (단 역함수는 결과의 범위가 정해져 있음) arc(아크)를 붙여 사용한다.

![ArcCosSin](/assets/images/post_img/gamemath/ArcCosSin.jpg)

--- 

## 자주 사용하는 공식

게임 좌표 연산에서 자주 활용되는 개념에 대해 알아보자.

- 코사인 법칙
    ![CosineLaw](/assets/images/post_img/gamemath/CosineLaw.jpg)
- 코사인 덧셈 정리
    - 회전 행렬에서 자주 사용된다. 
    ![CosineAdditionTheorem](/assets/images/post_img/gamemath/CosineAdditionTheorem.jpg)

&nbsp;

- 공식 활용 예시
    - 거리 계산
        ```cpp
        float distance = sqrt(dx * dx + dy * dy);
        ```
    - 두 벡터의 사이각 구하기
        ```cpp
        float dot = Normalize(a).Dot(Normalize(b));
        float angle = acos(dot);
        ```

---

## 마치며

오랜만에 삼각함수 개념을 공부했다. 바로 기억이 안나는 부분이 있어서 주기적으로 복습해야겠다!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*