---
title: "[DirectX 12] #5-1. Camera"
excerpt: "게임 수학 개념을 코드로 구현하기"

categories:
  - DirectX12
tags:
  - [DirectX, Camera]

permalink: /DirectX12/camera/

toc: true
toc_sticky: true

date: 2025-05-21
last_modified_at: 2025-05-21
---

## 들어가며

이번 시간에는 게임 수학 포스트에서 살펴봤던 개념을 실제 코드로 구현해본다. Transform 클래스와 좌표계 변환, 그리고 행렬 연산 등을 구현할 것이다.

---

## SIMD

DirectX12의 수학 연산에서 `XMMATRIX`는 SIMD(Single Instruction, Multiple Data)를 기반으로 작동하는데, 일반적인 float 연산보다 벡터와 행렬 연산에 최적화된 특수 레지스터를 활용한다. 그래서 직접 float 연산을 구현하기보다 전용 라이브러리를 사용하는 것이 좋다.

&nbsp;

- 설치 및 적용
    1. Google에 DirectXTK12 검색 후 다운로드
    2. 3개의 파일을 Engine 프로젝트에 추가 `Inc/SimpleMath.h`, `Inc/SimpleMath.inl`, `Src/SimpleMath.cpp`
    3. `Utils` 필터에 3개 파일을 드래그하여 포함시키기
    4. `EnginePch.h`에 헤더 추가 및 typedef 정의
        ```cpp
        #include "SimpleMath.h"

        using Vec2 = DirectX::SimpleMath::Vector2;
        using Vec3 = DirectX::SimpleMath::Vector3;
        using Vec4 = DirectX::SimpleMath::Vector4;
        using Matrix = DirectX::SimpleMath::Matrix;
        ```

---

## 흐름

- Local → World 좌표 변환에서의 계층 구조
    - 게임 오브젝트는 계층 구조(부모-자식)를 가질 수 있는데, 자식 객체는 부모의 Local을 원점으로 좌표를 갖게 된다.
    - 자식 객체는 Local → Local → World로 좌표계 변환을 한 단계 더 수행해야 한다.
    - 각 Transform이 자신의 Local 변환을 계산한 뒤, 부모의 World 변환 행렬을 곱하여 자신의 World 행렬을 구한다.

![HierarchyLocalCoordinate](/assets/images/post_img/directx/HierarchyLocalCoordinate.png)


- `FinalUpdate()`
    - `Component`, `GameObject`, `Scene`, `SceneManager` 클래스에 `FinalUpdate()`를 추가한다.
    - Object 별로 최종 위치, 회전, 스케일 변환을 수행하는 단계로, Engine 루프의 마지막에 호출된다.
    - 예. `Transform::FinalUpdate()`에서 SRT 순서로 변환 행렬을 계산하고, 계층 구조를 반영하여 _matWorld를 계산한다.

- `PushData()`
    - 최종 `World * View * Projection` 행렬을 계산헤 Shader에 전달한다.
    - `TransformParams` 구조체에 `matWVP`를 저장 후 Constant Buffer로 GPU에 넘긴다.
    - Shader에서는 이 행렬을 이용해 정점의 위치를 화면 좌표로 변환한다.

- Camera
    - 카메라는 View, Projection 행렬을 계산한다.
        - View: 카메라 Transform의 역행렬
        - Projection: 원근 or 직교 투영 설정에 따라 계산
    - 계산된 행렬은 전역 정적 변수(`S_MatView`, `S_MatProjection`)로 저장되어 다른 객체에서도 접근 가능하다.

- 렌더링 흐름 재정비
    - `MeshRenderer::Render()`에서는 Transform, Material, Mesh 각각의 데이터를 GPU에 전달한 뒤 실제로 Mesh를 그린다.
    - `Camera::Render()`에서는 Scene에 있는 GameObject를 순회하면서 MeshRenderer가 붙은 오브젝트만 렌더링한다.

- TestScene
    - TestObject 생성: Transform, MeshRenderer, Material 등 연결
    - Camera 오브젝트 생성: Transform, Camera, TestCameraScript 연결
    - TestCameraScript에서는 키보드 입력을 받아 카메라를 이동, 회전하도록 함.

- 결과
    - 키보드의 입력에 따라 카메라가 잘 움직인다.

![Result](/assets/images/post_img/directx/CameraResult.gif)

---

[Camera 커밋](https://github.com/chaeeun-dev/DirectX12/commit/a06b0a855de65285f8b74ab9f6b248e9c4b61318)

---

## 마치며

다렉은 나무보다는 숲을 봐야하는데, 블로그에 정리하면서 너무 시간도 오래걸리고 이해도 잘 안 되는 것 같아 과감하게 코드를 제외하고 흐름만 정리하는 방식으로 바꿨다. 코드는 깃허브에 올리니까 흐름만 잘 정리하자!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*