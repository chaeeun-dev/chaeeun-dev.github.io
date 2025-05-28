---
title: "[DirectX 12] #5-8. Frustum Culling"
excerpt: "절두체 컬링에 대한 이해"

categories:
  - DirectX12
tags:
  - [DirectX, Culling, FrutumCulling]

permalink: /DirectX12/frustum-culling/

toc: true
toc_sticky: true

date: 2025-05-28
last_modified_at: 2025-05-28
---

## 들어가며

저번 시간에는 반시계 방향의 면은 그리지 않는 Culling 방법에 대해 알아봤는데, 이번에는 Frustum Culling에 대해 알아볼 것이다.

---

## Frustum Culling

> 💡 Frustum(절두체)이란?
>
> - 카메라가 볼 수 있는 시야 범위를 3D 공간에 표현한 것.
> - 모양: 피라미드의 위를 잘라낸 형태

&nbsp;

- Frustum Culling
    - 렌더링 최적회를 위한 기술로, 카메라에 보이지 않는 오브젝트는 GPU에 넘기지 않고 CPU에서 미리 제외한다. (Skybox는 예외)
    - 차이점
        - 일반 렌더링: 모든 오브젝트를 GPU에 넘기고, GPU가 픽셀 단위까지 계산 후 화면 밖이면 그리지 않는다.
        - Frustum Culling: CPU가 먼저 Frustum 범위 바깥인지 판별 후 렌더링하지 않는다.
        - 성능 최적화에 매우 큰 효과가 있다!

---

### 수학 증명

- 평면식과 점의 위치 증명 

![증명](/assets/images/post_img/directx/FrustumMath.jpg)

---

### 게임 응용

- Frustum 판별하기
    1. Frustum 6개 평면의 Normal 벡터를 구한다.
    2. 점이 각 평면의 앞/뒤 중 어디에 위치하는지 계산한다. (`ax + by + cz + d`에 점을 대입)
    3. 모든 평면에 대해 음수 값이라면, Frustum 안에 있으므로 렌더링한다.

&nbsp;

- Transform
    - 오브젝트 좌표는 일반적으로 `Local → World → View → Projection` 이 단계를 거쳐 좌표 변환이 된다.
    - Frustum은 Projection 공간에 존재하기 때문에, World 공간으로 역변환해서 비교해야 한다.

&nbsp;

- Frustum 면 생성
    - 각 면의 Normal 방향은 Frustum의 바깥쪽을 향해야 한다.
        ![Frustum](/assets/images/post_img/directx/FrustumCube.jpg)
- Sphere Culling
    - 구의 중심과 반지름을 사용한다.
    - 평면과 구의 중심 사이의 거리를 계산한다. 반지름보다 크면 평면 뒤에 있다. 
- 라이브러리 활용
    - `BoundingFrustum::Intersects(BoundingSphere sphere);`
    - DirectX에서 이와 같은 함수를 제공하는데데, 수학 계산 없이 간단하게 컬링할 수 있다.

---

[Frustum Culling 커밋](https://github.com/chaeeun-dev/DirectX12/commit/7835b84b6b889a3afe8f89253f1bdb959ba3dd6c)

---

## 마치며

CPU 단계에서 바로 컷해버리니까 엄청난 성능 연산이 될 수 밖에 없구나. 게임 성능 향상을 위한 방법이 정말 많은 것 같다. 그 중 절두체 컬링이 가장 효과적인 것 같다!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*

[Frustum Culling](https://gamedev.net/tutorials/programming/general-and-gameplay-programming/)