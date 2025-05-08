---
title: "[GameMath] 행렬"
excerpt: "행렬에 대한 이해와 게임에서의 활용"

categories:
  - GameMath
tags:
  - [GameMath, Matrix]

permalink: /GameMath/matrix/

toc: true
toc_sticky: true

date: 2025-05-07
last_modified_at: 2025-05-09
---

## 들어가며

이번 시간에는 행렬에 대해 알아본다. 게임에서 행렬은 벡터 변환과 같은 다양한 연산에 필수적인 개념이다. 인공지능, 물리, 카메라 등에서도 굉장히 많이 사용하기 때문에 정말 중요하다.

---

## 행렬

- 행렬이란?
    - 수를 직사각형 형태로 배열한 것으로, 행과 열로 구성되어 있다.
    - 행(Row): 가로 줄
    - 열(Column): 세로 줄

---

### 행렬의 연산

- 덧셈과 뺄셈
    - 두 행렬의 각 원소끼리 더하거나 뺀다.
    - 단, 행과 열의 크기가 같아야 한다.
- 스칼라 곱
    - 행렬의 각 원소에 스칼라를 곱한다.
- 행렬의 곱셈
    - 조건: A (m×n) 행렬, B (n×p) 행렬일 때만 곱셈 가능하다.
    - 결과: C (m×p) 행렬
    - 교환법칙은 성립하지 않는다. `AB ≠ BA`
    - 결합법칙은 성립한다. `(AB)C = A(BC)` → 변환의 순서가 중요할 때 매우 중요

![MatrixBasicOperation](/assets/images/post_img/gamemath/MatrixBasicOperation.jpg)

---

### 행렬의 종류

- 대각 행렬 (Diagonal Matrix)
    - 대각선 요소를 제외한 모든 원소가 0인 행렬이다.
- 단위 행렬 (Identity Matrix)
    - 대각선 요소는 1, 나머지 원소는 모두 0인 행렬이다.
    - 특징: `A * I = A`, `I * A = A`
    - 벡터나 행렬에 아무런 영향을 주지 않는다.
- 역행렬 (Inverse Matrix)
    - 행렬 A의 역행렬 B는 `AB = I`를 만족한다.
    - 역행렬이 존재하지 않는 경우도 있다. 역행렬이 존재하면 가역행렬이라고 부른다.
    - 조건: `det(A) ≠ 0`이어야 역행렬이 존재한다.
    - 고차원 행렬일수록 계산이 복잡하다.
- 전치 행렬 (Transpose Matrix)
    - 행과 열을 뒤집은 행렬이다.
    - 예: `Aᵗ = Transpose(A)`
    - 특징: 어떤 행렬 M에 대해 `M * Mᵗ = I`인 경우도 있다.
- 직교 행렬 (Orthogonal Matrix)
    - 벡터들이 서로 직각을 이루는 행렬이다.
    - 특징: `Mᵗ = M⁻¹` (전치 행렬 = 역행렬)
    - 회전 행렬은 대부분 직교행렬이다.

![MatrixVariety](/assets/images/post_img/gamemath/MatrixVariety.jpg)

---

## 마치며

오랜만에 행렬을 복습하니 예전 기억이 떠오른다.. 직교 행렬 부분이 좀 헷갈리니 복습해야지!!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*