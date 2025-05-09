---
title: "[GameMath] 좌표계 변환 행렬"
excerpt: "좌표계 변환 행렬에 대한 이해"

categories:
  - GameMath
tags:
  - [GameMath, CoordinateTransformation, Matrix]

permalink: /GameMath/coordinate-transformation-matrix/

toc: true
toc_sticky: true

date: 2025-05-09
last_modified_at: 2025-05-09
---

## 들어가며

이번 시간에는 좌표계 변환 행렬에 대해 알아본다. 좌표계 변환은 특히 월드 좌표계(World Space)에 존재하는 객체를 카메라 좌표계(View Space)로 변환하거나, 모델 좌표계(Model Space)를 월드 좌표계로 변환할 때 자주 사용되는 개념이다.

---

## 좌표계 변환 행렬 

2차원에서 A 좌표계의 좌표 M `(x1, y1)`을 B 좌표계의 좌표 `(x2, y2)`로 변환해보자.

![MatrixCoordinateTransformation](/assets/images/post_img/gamemath/MatrixCoordinateTransformation.jpg)

&nbsp;

위의 식을 토대로 3차원 좌표계 변환 행렬을 계산하면 다음과 같은 4x4 행렬(회전 + 이동)이 나온다.

![MatrixCoordinateTransformationLaw](/assets/images/post_img/gamemath/MatrixCoordinateTransformationLaw.jpg)

- 각 요소들
    - 상단의 3x3 요소 (회전 행렬)
        - 각 열은 대상 좌표계의 x, y, x 축이 기존 좌표계에서 어떻게 배치되어 있는지를 나태낸다.
        - 각 행은 기존 좌표계의 축을 대상 좌표계에서 어떻게 바라보는지를 나타낸다.
    - 우측 열 (이동 벡터)
        - 새로운 좌표계의 원점이 기존 좌표계에서 어디로 이동했는지를 나타낸다.
    - 마지막 행 (동차 좌표계)
        - 동차 좌표계를 유지하기 위한 형식이다.

- 위치 vs 방향 벡터
    - 방향 벡터는 단순히 방향만 나타내기 때문에 이동이 반영되면 안 된다. 따라서 마지막 원소가 0이 되어 translation 열이 무시되도록 `(x, y, z, 0)`으로 표현된다.

---

## 마치며

이번 시간에는 좌표계 변환 행렬에 대해 공부했다. 중간에 이해가 안 돼서 좀 헤맸지만 천천히 식을 써보니 조금씩 이해한 것 같다.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*