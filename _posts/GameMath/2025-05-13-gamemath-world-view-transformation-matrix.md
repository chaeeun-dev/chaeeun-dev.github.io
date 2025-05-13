---
title: "[GameMath] World, View 변환 행렬"
excerpt: "World, View 좌표계 변환 이해하기"

categories:
  - GameMath
tags:
  - [GameMath, WorldSpace, ViewSpace, Transformation]

permalink: /GameMath/world-view-transformation-matrix/

toc: true
toc_sticky: true

date: 2025-05-13
last_modified_at: 2025-05-13
---

## 들어가며

저번 포스트에서는 좌표계 변환 행렬에 대해 알아보았다. 이번에는 좌표계 변환 행렬의 개념을 가지고 3D 모델을 화면에 렌더링하기 위해 좌표를 변환하는 여러 단계를 알아본다.

---

## 좌표계 변환

- 좌표 공간의 종류
    - Local Space 
        - 모델이 생성될 때의 좌표계로, 각 모델은 ㅜ 자기만의 기준 위치와 방향을 갖는다.
    - World Spcae
        - 각 모델이 실제 게임 월드에 배치되는 공간이다.
        - 위치(Position), 크기(Sclae), 회전(Rotation) 등의 변환이 적용되어 배치된다.
    - View Space(Camera Space, Eye Space)
        - 카메라의 시점에서 바라보는 좌표계이다.
        - 카메라는 고정되어 있고, 세상이 움직이는 것처럼 표현된다.
    - Projection Space
        - View Space를 정규화된 Clip Space 공간으로 투영한다.
        - 좌표는 `-1 ~ 1` 사이의 값으로 변환된다.
    - Screen Space
        - 실제 모니터의 픽셀 단위로 변환된 최종 좌표이다.

&nbsp;

>- 좌표계 변환
>   - Local Space → World Space → View Space → Projection Space → Screen Space

이번 시간에는 World 변환(Local → World)과 View 변환(World Space → View Space)에 대해 알아본다.

---

### World 변환 

World 변환은 모델의 Local Space 좌표를 World Space로 변환하는 과정이다. Unity Engine에서는 모델 변환 행렬이라고 부른다.

![WorldTransformation](/assets/images/post_img/gamemath/MatrixWorldTransformation.jpg)

> 💡 기저 벡터란>
> 
> Rotation 변환에서는 보통 모델의 기준 축 벡터인 Right, Up, Look을 사용하는데, 이처럼 서로 수직인 단위 벡터의 집합을 기저 벡터라고 부른다.

---

### View 변환

View 변환은 World 좌표계를 카메라 좌표계로 변환하는 과정이다. 카메라는 원점에 고정하고, 월드 상의 객체들을 카메라 기준으로 재배치하는 것이다.

&nbsp;

View 행렬은 카메라의 World 변환 행렬의 역행렬이다.

![ViewTransformation](/assets/images/post_img/gamemath/MatrixViewTransformation.jpg)

---

## 마치며

이번 시간에는 여러 좌표계 변환 단계 중 World와 View 변환에 대해 알아봤다. 저번 좌표계 변환 시간에 아리송했던 것들이 이해가 됐다! 3D 게임에서 정말 중요한 개념이니 꼭 숙지하자.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*