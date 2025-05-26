---
title: "[DirectX 12] #5-7. SkyBox"
excerpt: "레스터라이저 단계에서의 컬링에 대한 이해와 SkyBox"

categories:
  - DirectX12
tags:
  - [DirectX, Culling]

permalink: /DirectX12/skybox/

toc: true
toc_sticky: true

date: 2025-05-26
last_modified_at: 2025-05-26
---

## 들어가며

이번 시간에는 게임 공간에 하늘 배경을 그려본다.

---

## SkyBox

> 💡 SkyBox란?
>
> - 3D 공간의 하늘 배경을 그리는 기법으로, 매우 큰 구 또는 직육면체를 카메라 주위에 배치하여 하늘을 표현한다.   

&nbsp;

- 특징
    - 카메라
        - 카메라가 이동해도 똑같이 보여야 한다.
        - 카메라가 회전하면 하늘이 도는 것처럼 보여야 한다.
    - 깊이값
        - 항상 맨 뒤에 있어야 하므로 깊이 값은 1로 고정된다.
    - 컬링
        - 하늘은 내부에서 보는 구조이기 때문에 Back Face Culling을 Off 해야 한다.
    - 좌표계 변환
        - Local → View로 바로 변환한다. (World 변환 생략)
    - 투영 방식
        - View 변환 시 Translation은 제외, Rotation만 적용한다.

&nbsp;

- 정점 쉐이더

```hlsl
VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    // 뷰 행렬에 회전만 적용 ( w = 0 → Translation 무시)
    float4 viewPos = mul(float4(input.localPos, 0), g_matView);
    float4 clipSpacePos = mul(viewPos, g_matProjection);

    // 깊이 값을 1로 고정
    output.pos = clipSpacePos.xyww;
    output.uv = input.uv;

    return output;
}
```

&nbsp;

- 텍스처
    - 구 형태의 SkyBox
        - Google → `high resolution space hdri` 검색
        - 한 장의 equirectangular HDR 이미지가 필요하다. 
    - 직육 면체의 SkyBox
        - Google → `skybox texture` 검색
        - 6장의  텍스처가 필요하다. 
- → 이번에는 구 형태를 사용하는데 사각형 Texture를 사용하기 때문에 끊기는 부분이 있다. equrectangular 이미지를 사용하거나 직육면체로 수정하면 된다!

---

### Culling

일반적으로 정점은 시계방향으로 그려지는데, Rasterizer에서 Back Face Culling을 사용하면 반시계 방향은 그리지 않는다. 그런데 SkyBox는 내부에서 바깥면을 보는 구조이기 때문에 Cull Mode를 None 또는 Front로 설정해야 내부 면이 렌더링 된다. 

---

- 결과
    ![SkyBox](/assets/images/post_img/directx/SkyBoxResult.png)

---

[SkyBox 커밋](https://github.com/chaeeun-dev/DirectX12/commit/614a9176f6d3b4719e3e850249c5255a7f1c4092)

---

## 마치며

유니티 엔진에서 하늘을 어떻게 그리는지 궁금했는데, 이번 시간에 확실히 알게 되었다! 생각보다 간단하게 구현할 수 있었구나..!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*