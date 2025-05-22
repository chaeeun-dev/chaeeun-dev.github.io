---
title: "[DirectX 12] #5-3. Lighting"
excerpt: ""

categories:
  - DirectX12
tags:
  - [DirectX, Lighting]

permalink: /DirectX12/lighting/

toc: true
toc_sticky: true

date: 2025-05-22
last_modified_at: 2025-05-22
---

## 들어가며

그래픽에서 물체를 사실적으로 표현하기 위해 조명(Lighting)은 매우 중요하다. 이번 시간에는 빛의 종류와 물리적 성질을 수학적으로 알아본다.

---

## 조명

조명은 빛의 방향성과 범위에 따라 여러 종류로 나뉜다.

&nbsp;

- Directional Light
  - 태양처럼 무한한 거리에서 비추는 빛이다.
  - 주요 요소: 방향(Direction)
- Point Light
  - 한 지점에서 모든 방향으로 퍼지는 빛이다.
  - 주요 요소: 위치(Position), 최대 거리(Range)
- Spot Light
  - 손전등처럼 특정 방향으로 퍼지는 빛이다.
  - 주요 요소: 위치(Position), 거리(Range), 각도(Angle)

&nbsp;

- SpotLight에서 정점의 위치 판별하기

![SpotLight](/assets/images/post_img/directx/LightingSpotLight.jpg)

---

## 빛

빛은 물체와 상호작용할 때 세 가지 성분으로 나뉘는데, 이 성분의 합으로 물체의 최종 색상이 결정된다.

&nbsp;

- Ambient(환경)
  - 그림자 속에서도 간접적으로 받는 빛이다. 
  - 예. 책상 밑에서도 약간의 빛이 보임
- Diffuse(난반사)
  - 빛이 물체 표면에서 여러 방향으로 퍼진다. 일반적인 밝기 느낌이다.
- Specular(정반사)
  - 금속처럼 표면이 매끄러울 때 생기는 하이라이트 효과이다.

![AmbientDiffuseSpecular](/assets/images/post_img/directx/AmbientDiffuseSpecular.png)

&nbsp;

- Diffuse와 Specular 구하는 방법 (Ambient는 전체적으로 더해주기 때문에 구하지 않아도 됨)

![DiffuseSpecular](/assets/images/post_img/directx/LightingDiffuseSpecular.jpg)

&nbsp;

최종적으로, `(Diffuse + Ambient + Specular) * ObjectColor` 수식으로 물체의 색상이 결정된다!

---

## 마치며

조명의 종류와 빛의 세 가지 성분에 대해 알아보았다. 삼각함수로 모든 계산을 할 수 있는 게 신기하고 재밌다 ㅎㅎ

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*

*[Ambient, Diffuse, Specular](https://clara.io/learn/user-guide/lighting_shading/materials/material_types/webgl_materials)*