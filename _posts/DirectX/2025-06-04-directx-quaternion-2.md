---
title: "[DirectX 12] #6-2. Quaternion 2"
excerpt: "쿼터니언 계산하기"

categories:
  - DirectX12
tags:
  - [DirectX, Quaternion]

permalink: /DirectX12/quaternion-2/

toc: true
toc_sticky: true

date: 2025-06-04
last_modified_at: 2025-06-04
---

## 들어가며

이번 시간에는 저번 시간에 배운 복소수에 이어 쿼터니언에 대한 기초에 대해 알아볼 것이다.

---

## Quaternion

- 쿼터니언의 기본 개념
  ![QuaternionBasic](/assets/images/post_img/directx/QuaternionBasic.jpg)

&nbsp;

- 쿼터니언의 곱셉
  ![QuaternionMultiply](/assets/images/post_img/directx/QuaternionMultiply.jpg)

  &nbsp;

- 쿼터니언 계산
  ![QuaternionCalculate](/assets/images/post_img/directx/QuaternionCalculate.jpg)

&nbsp;

- 쿼터니언 회전
  ![QuaterniobRoation](/assets/images/post_img/directx/QuaternionRotation.jpg)

&nbsp;

- 쿼터니언의 행렬 표현
  1. 쿼터니언 → 행렬
  2. 행렬 → 쿼터니언

  ![QuaternionMatrix](/assets/images/post_img/directx/QuaternionMatrix.jpg)


## 마치며

증명이 너무 복잡해서 제일 어려운 수업이었다. 그치만 결과는 매우 간단해서 코드 구현은 쉬울 것 같다.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*