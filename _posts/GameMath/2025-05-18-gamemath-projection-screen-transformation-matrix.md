---
title: "[GameMath] Projection, Screen 변환 행렬"
excerpt: "Projection, Screen 변환 행렬 이해하기"

categories:
  - GameMath
tags:
  - [GameMath, Projection, Screen, Transformation]

permalink: /GameMath/projection-screen-transformation-matrix/

toc: true
toc_sticky: true

date: 2025-05-18
last_modified_at: 2025-05-18
---

## 들어가며

저번 포스트에서는 좌표계 변환 중 World 변환과 View 변환에 대해 알아봤다. 이번에는 Projection 변환과 Screen 변환에 대해 알아본다. 

---

## 좌표계 변환

> Local Space → World Space → View Space → Projection Space → Screen Space

---

### Projection 변환

투영 변환은 3D에 존재하는 점을 2D로 변환하는 것이고, 원근 투영과 직교 투영으로 나뉜다. 여기서는 게임 수학에서 자주 사용하는 원근 투영 변환 행렬에 대해 알아본다.

&nbsp;

- n(near): 원점에서 가까운 평면까지의 거리
- f(far): 원점에서 먼 평면까지의 거리
- r: 화면비(w/h)

![ProjectionTransformation](/assets/images/post_img/gamemath/MatrixProjectionTransformation.jpg)

---

### Screen 변환

Projection 변환을 통해 각 좌표들은 `-1 ~ 1`의 값으로 세팅되었다. 이제 Screen 변환을 통해 화면 좌표로 변환을 해보자.

![ScreenTransformation](/assets/images/post_img/gamemath/MatrixScreenTransformation.jpg)

---

## 마치며

여러 식들과 행렬로 굉장히 헷갈렸던 수업이었다.. 아직 완벽하게 이해하진 못한 것 같지만 다른 블로그도 찾아보면서 이해해야겠다. 이번 포스트로 DirectX12 수업에서의 게임 수학 포스트는 끝!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*