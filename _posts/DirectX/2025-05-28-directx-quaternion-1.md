---
title: "[DirectX 12] #6-1. Quaternion 1"
excerpt: "오일러 회전의 문제점과 복소수 & 쿼터니언에 대한 이해"

categories:
  - DirectX12
tags:
  - [DirectX, Quaternion]

permalink: /DirectX12/quaternion-1/

toc: true
toc_sticky: true

date: 2025-05-28
last_modified_at: 2025-05-28
---

## 들어가며

이번 시간에는 오일러 회전 방식의 회전의 문제점에 대해 알아보고, 문제를 해결할 쿼터니언 회전 방식에 대해 간략하게 알아볼 것이다. 쿼터니언을 이해하기 위해 복소수를 자세히 알아본다. 

---

## 회전

- 오일러 회전
    - x, y, z축을 기준으로 순차적으로 회전한다.
    - Unity의 기본 회전 순서는 z, x, y 순이다.
    - 문제점: 짐벌락(Gimbal Lock)
        - 서로 다른 축을 순서대로 회전하다보면, 두 축이 겹쳐 하나의 축이 사라지는 현상이 발생할 수 있다.
        - 예. x축을 90도 회전시키면 y, z축이 정렬되어 하나의 축이 먹통이 된다.
- Quaternion 회전
    - 게임 엔진은 내부적으로 회전 계산에 쿼터니언을 사용하는데, 툴 UI는 오일러 방식으로 표시된다. 
    - 이번 시간에는 쿼터니언을 이해하기 위해 복소수에 대해 자세히 알아볼 것이다. 다음 포스트에 자세히 다룰 예정!

---

### 복소수

![ComplexNumber](/assets/images/post_img/directx/QuaternionComplexNumber.jpg)

&nbsp;

![PolarCoordinate](/assets/images/post_img/directx/QuaternionPolarCoordinate.jpg)

- 복소수는 `a + bi` 형태로 2D 평면의 회전을 표현한다.
- Quaternion은 이를 3D로 확장시켜 3D 회전을 표현할 수 있다.

---

## 마치며

중학교인지 고등학교인지..? 예전에 뭐에 쓰이는지도 모르고 배운 복소수로 3D 그래픽의 회전을 표현하는 게 새롭다!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*