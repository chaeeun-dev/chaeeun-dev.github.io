---
title: "[DirectX 12] #2-6. Index Buffer"
excerpt: "vertex buffer와 index buffer로 효율적으로 Mesh 그리기"

categories:
  - DirectX12
tags:
  - [DirectX, Mesh, VertexBuffer, IndexBuffer]

permalink: /DirectX12/index-buffer/

toc: true
toc_sticky: true

date: 2025-03-28
last_modified_at: 2025-03-28
---

## 들어가며

DX에서 3D 모델을 사용할 때, 같은 객체를 여러 번 생성한다고 해서 매번 메쉬 정보를 다시 로드하지 않는다. 한 번 로드된 메쉬 데이터를 여러 객체가 공유하여 성능을 최적화한다.

이전 코드에서는 다음과 같은 방식으로 렌더링했다.
1. Mesh를 생성할 때 정점 정보를 저장
2. vertex buffer를 생성하고 vertex buffer view를 생성
3. Input Assembler에 vertex buffer view 전달
4. 쉐이더에서 정점 정보를 받아 연산 수행
5. 삼각형 단위로 화면에 렌더링

![Image](https://github.com/user-attachments/assets/472676b2-1154-41e6-880f-b2dd3e6dd5b3)

이번 수업에서는 vertex buffer와 index buffer를 함께 사용해서 Input Assembler에 전달하여 사각형을 렌더링 하는 방법을 배운다.

---

## 사각형 그리기

vertex buffer만 사용해서 사각형을 그리는 방식과 vertex buffer + index buffer를 사용해 사각형을 그리는 방식을 코드로 구현해보자.

![Image](https://github.com/user-attachments/assets/e2a1cc14-5b1d-4713-81de-10fe86654752)

---

### vertex buffer only

vertex buffer만 사용할 경우, 사각형을 두 개의 삼각형으로 나눠 총 6개의 정점을 저장해야 한다.

&nbsp;

- 정점 데이터 수정 (`Clinet` 프로젝트의 `Game::Init()`)
    - 정점 데이터 6개
```cpp
vector<Vertex> vec(6);
vec[0].pos = Vec3(-0.5f, 0.5f, 0.5f);
vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);
vec[1].pos = Vec3(0.5f, 0.5f, 0.5f);
vec[1].color = Vec4(0.f, 1.f, 0.f, 1.f);
vec[2].pos = Vec3(0.5f, -0.5f, 0.5f);
vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);

vec[3].pos = Vec3(0.5f, -0.5f, 0.5f);
vec[3].color = Vec4(0.f, 0.f, 1.f, 1.f);
vec[4].pos = Vec3(-0.5f, -0.5f, 0.5f);
vec[4].color = Vec4(0.f, 1.f, 0.f, 1.f);
vec[5].pos = Vec3(-0.5f, 0.5f, 0.5f);
vec[5].color = Vec4(1.f, 0.f, 0.f, 1.f);

mesh->Init(vec);
---

- 결과
    - 6개의 정점을 사용하여 삼각형 두 개가 생성됐고, 사각형이 출력 됐다.
- 문제점
    - 중복된 정점이 있어 메모리의 낭비가 발생한다.
    - index buffer를 사용하면 정점 데이터 낭비를 줄일 수 있다.

---

### vertex buffer + index buffer

index buffer에는 정점을 그리는 순서를 저장하고, vertex buffer에는 정점의 정보를 저장하여 정점 정보가 중복 저장되는 문제점을 해결할 수 있다.

&nbsp;

- 정점 데이터 수정 (`Clinet` 프로젝트의 `Game::Init()`)
    - vertex 데이터 4개
        ```cpp
        vector<Vertex> vec(4);
        vec[0].pos = Vec3(-0.5f, 0.5f, 0.5f);
        vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);
        vec[1].pos = Vec3(0.5f, 0.5f, 0.5f);
        vec[1].color = Vec4(0.f, 1.f, 0.f, 1.f);
        vec[2].pos = Vec3(0.5f, -0.5f, 0.5f);
        vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);
        vec[3].pos = Vec3(-0.5f, -0.5f, 0.5f);
        vec[3].color = Vec4(0.f, 1.f, 0.f, 1.f);
        ```
    - index 데이터
        ```cpp
        vector<uint32> indexVec;
        indexVec.push_back(0); indexVec.push_back(1); indexVec.push_back(2);
        indexVec.push_back(0); indexVec.push_back(2); indexVec.push_back(3);
        ```

&nbsp;

- `Mesh` 클래스 수정
    - index buffer 지원을 위해 index buffer 관련 멤버 변수를 추가한다.
        ```cpp
        ComPtr<ID3D12Resource>		_indexBuffer;
        D3D12_INDEX_BUFFER_VIEW		_indexBufferView;
        uint32 _indexCount = 0;
        ```
    - index buffer를 생성하는 `Mesh::CreateIndexBuffer()`를 생성한다.
        ```cpp
        _indexCount = static_cast<uint32>(buffer.size());
        uint32 bufferSize = _indexCount * sizeof(uint32);

        D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
        D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

        DEVICE->CreateCommittedResource(
            &heapProperty,
            D3D12_HEAP_FLAG_NONE,
            &desc,
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&_indexBuffer));

        void* indexDataBuffer = nullptr;
        CD3DX12_RANGE readRange(0, 0);
        _indexBuffer->Map(0, &readRange, &indexDataBuffer);
        ::memcpy(indexDataBuffer, &buffer[0], bufferSize);
        _indexBuffer->Unmap(0, nullptr);

        _indexBufferView.BufferLocation = _indexBuffer->GetGPUVirtualAddress();
        _indexBufferView.Format = DXGI_FORMAT_R32_UINT;
        _indexBufferView.SizeInBytes = bufferSize;
        ```
    - 기존의 `Mesh::Init()` 수정
        - 기존의 코드는 `CreateVertexBuffer()`로 바꾼다.
        - `CreateVertexBuffer()`와 `CreateIndexBuffer()`를 호출한다.
    - `Mesh::Render()` 수정
        ```cpp
        CMD_LIST->IASetIndexBuffer(&_indexBufferView);
        CMD_LIST->DrawIndexedInstanced(_indexCount, 1, 0, 0, 0);
        ```

- 결과
    - 똑같은 사각형이 출력되지만, 정점을 그리는 방식에 차이가 있다.

&nbsp;


- 차이점
    - vertex buffer만 사용할 경우
        - 정점이 6개 필요하다.
        - 중복된 점으로 메모리가 낭비된다.
    - vertex buffer와 index buffer를 같이 사용한 경우
        - 정점이 4개 필요하다.
        - 메모리를 절약해 렌더링할 수 있다.
        - 고퀄리티 3D 모델을 다룰 때 큰 성능 차이를 보인다.

---

[Index Buffer 커밋](https://github.com/chaeeun-dev/DirectX12/commit/77d213d19cb2989ea885449b426ace3884cf997c)

---

## 마치며

이번 시간에 배운 것
    - vertex buffer만 사용해서 삼각형을 그리는 방법
    - vertex buffer와 index buffer를 사용하여 효율적으로 렌더링하는 방법

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*