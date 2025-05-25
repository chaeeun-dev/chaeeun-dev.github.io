---
title: "[DirectX 12] #5-5. Normal Mapping"
excerpt: "Normal Mapping에 대한 필요성과 이해"

categories:
  - DirectX12
tags:
  - [DirectX, NormalMapping]

permalink: /DirectX12/normal-mapping/

toc: true
toc_sticky: true

date: 2025-05-25
last_modified_at: 2025-05-25
---

## 들어가며

Cube에 Texture를 매핑했을 때 너무 밋밋한 문제가 있다. 이 문제를 해결하기 위해 Normal Mapping 개념이 사용된다.

> Texture 로드하기
>
> [3D Texture](https://3dtextures.me/ )
> 사이트에서 원하는 이미지 normal, base color 다운 후 Resource/Texture 경로에 넣기

---

## Normal Mapping

- 기존 방식의 문제점
    - 기본적으로 삼각형의 각 꼭짓점에 Normal을 주는데, 삼각형 내부의 픽셀들은 보간된 Normal을 사용한다.
    - 정점 수가 적으면(Sphere보다 Cube가 훨씬 적음) 내부의 Normal이 너무 균일해서 밋밋한 조명 결과가 나타난다.
- 해결책
    - 정점을 늘리는 방식은 메모리와 성능 측면에서 매우 비효율적이다.
    - Normal Texture를 사용해서 픽셀마다 고유한 Normal 벡터를 사용하도록 한다.

&nbsp;

- Normal Map
    - RGB 이미지로 구성: 각 픽셀에 XYZ 방향 벡터를 담고 있다. 좌표계는 Tangent Space이다.
        - R: Tangent(x축)
        - G: Binormal(y축)
        - B: Normal(z축)
    - 파란색: 대부분의 픽셀이 (0, 0, 1) 근처의 벡터인데, 표면을 정면으로 바라보는 Normal이기 때문이다.
    - 정규화: 각 값은 `0 ~ 255`에서 `-1 ~ 1` 범위로 정규화가 필요하다.

&nbsp;

> 💡 Tangent Space
>
> - Mesh의 표면 기준으로 정의된 공간이다.
>   - X축: Tangent
>   - Y축: Binormal
>   - Z축: Normal
> - 이 공간은 Mesh가 회전하거나 움직여도 함께 회전한다.

&nbsp;

- Tangent 공간 → View 공간 변환
    - 이 프로젝트에서는 조명 계산을 View Space에서 하기 떄문에, Normal Map의 벡터를 View Space로 변환해야 한다.
    - 프로젝트에서는 Tangent와 Normal 값을 이용해 Binormal 값을 계산하는데, 오른손/왼손 좌표계에 따라 Binormal 값이 음수로 될 수 있다는 점 주의하기!

![TangentSpace](/assets/images/post_img/directx/TangentSpace.jpg)

&nbsp;

- 결과
    - Normal Mappping을 적용하지 않았을 떄
        ![NonNormalMapping](/assets/images/post_img/directx/NormalMappingNonResult.png)
    - Normal Mapping을 적용했을 때
        ![NormalMapping](/assets/images/post_img/directx/NormalMappingResult.png)
    - Normal Mapping을 적용한 결과가 훨씬 입체적이고 사실적이다.
    
---

[Normal Mapping 커밋](https://github.com/chaeeun-dev/DirectX12/commit/7e7bd85dbbc14d9376339d22ba1b972cd6ddd47c)

---

## 마치며

Normal Mapping에 대해 알아보았다. 픽셀에 Normal 값만 적용해줘도 그래픽적으로 훨씬 사실적이라는 게 신기했다. 정점 개수를 늘리지 않고 텍스처 하나를 추가해서 만든다는 게 놀랍다.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*