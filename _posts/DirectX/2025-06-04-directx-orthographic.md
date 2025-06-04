---
title: "[DirectX 12] #7-1. 직교 투영"
excerpt: "원근 투영과 직교 투영 이해하고 카메라로 구현하기"

categories:
  - DirectX12
tags:
  - [DirectX, Projection, Orthographic]

permalink: /DirectX12/orthographic/

toc: true
toc_sticky: true

date: 2025-06-04
last_modified_at: 2025-06-04
---

## 들어가며

이번 시간에는 UI애 자주 사용되는 직교 투영에 대해 알아보고 카메라로 구현해본다.

---

## 투영

![Projection](/assets/images/post_img/directx/Projection.png)

- 원근 투영(Perspective)
    - 멀면 작게, 가까우면 크게 보인다.
    - 3D 게임 회면, 3D UI에 적용된다.
- 직교 투영(Orthographic)
    - 거리에 상관 없이 항상 같은 크기가 적용된다.
    - UI, 인벤토리 등
    - 시야각과 깊이 값의 개념이 없다.
- 예시
    - 위: 원근 투영
    - 아래: 직교 투영
    ![GameUI](/assets/images/post_img/directx/ProjectionUI.png)

---

### 직교 투영 행렬

화면의 비율에 맞춰 변환하면 된다. 원근 투영보다 훨씬 간단하다.

![ProjectionMatrix](/assets/images/post_img/directx/ProjectionOrthographicMath.jpg)

---

## 구현

- 쉐이더 확장자 변경
    - `.hlsli`을 `.fx`로 변경한다.
    - 속성 → HLSL 컴파일러
        - 쉐이더 모델: `Shader Model 5.0`
        - 쉐이더 유형: `쉐이더 효과 (/fx)`
- 카메라 분리하기
    - 직교 투영 카메라와 원근 투영 카메라 2개로 동시에 찍는다.
    - 각각의 카메라는 다른 Layer만 렌더링하도록 설정한다.
- Layer 시스템
    - 각 GameObject는 Layer 정보를 가진다. (`uint8`로 32개)
    - 카메라는 어떤 Layer를 렌더링할지 설정된다.
    - 예시.
        - 직교 카메라: UI 전용 Layer만 렌더링
        - 원근 카메라: 일반 3D Object Layer만 렌더링한다.
    - Unity Engine은 32개의 Layer가 존재하며, 각 카메라는 특정 Layer만 그리도록 설정할 수 있다. 

---

[직교 투영 커밋](https://github.com/chaeeun-dev/DirectX12/commit/1ca4ee0348ea57b018eeed360fed2834e8f99011)

---

## 마치며

2D 프로젝트에서 UI 렌더링을 할 때 그냥 Layer 개념만 가지고 맨 위에 그렸는데, 이 수업에서는 UI 전용 카메라를 하나 더 두어 UI만 찍도록 하는 게 신기했다. 3D 게임 프로젝트 시작하면 카메라를 하나 더 만들어야겠다!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*

*[Projection](https://www.researchgate.net/figure/Projection-modes-aorthographic-projection-and-bperspective-projection_fig2_265693452)*