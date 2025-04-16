---
title: "[DirectX 12] #2-8. DepthStencilView"
excerpt: "카메라 좌표계를 투영 좌표게로 변환하기"

categories:
  - DirectX12
tags:
  - [DirectX, DepthStencilView]

permalink: /DirectX12/depth-stencil-view/

toc: true
toc_sticky: true

date: 2025-04-16
last_modified_at: 2025-04-16
---

## 들어가며

이번 시간에는 Depth 개념을 알아보고, 픽셀의 깊이 값을 추적해 가장 가까운 객체의 픽셀만 출력되도록 처리하는 실습을 한다.

---

## Depth

3D 게임의 세상은 입체적이지만, 모니터에 출력되는 화면은 2D로 납작하다. 3D를 2D로 나타내기 위해 투영(Projection)이 필요하다.

> 💡 Projection이란?
>
> 카메라 좌표계를 투영 좌표계로 변환해 원근감을 적용한다.

&nbsp;

- Depth Buffer
    - 역할
        - 픽셀 단위로 깊이 값을 저장한다.
        - 가장 가까운 픽셀만 화면에 출력한다.
    - 예시
        - 두 사각형이 겹칠 경우, 카메라에 가까운 사각형만 보인다.
    - 특징
        - 깊이 값은 일반적으로 0.0 ~ 1.0이다.
        - 1에 가까울 수록 멀리 있는 오브젝트이고, 1 이상은 아예 계산하지 않는다.


---

## Stencil

실습에서 사용하지 않지만, 개념적으로 알아보자.

- Stencil
    - 개념
        - 미술에서 판에 구멍을 뚫고 잉크를 통과시켜 찍어내는 공판화 기법이다.
        - 그래픽스에선 픽셀마다 스텐실 값을 기록하고, 특정 조건을 만족할 때만 해당 픽셀에 렌더링을 허용한다.
    - 활용 예시
        - 실루엣 처리, 미러 효과, 그림자 마스킹 등

---


---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*