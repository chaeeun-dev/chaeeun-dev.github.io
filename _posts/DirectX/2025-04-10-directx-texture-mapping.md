---
title: "[DirectX 12] #2-7. Texture Mapping"
excerpt: "Mesh에 Texture 입히기"

categories:
  - DirectX12
tags:
  - [DirectX, Texture]

permalink: /DirectX12/texture-mapping/

toc: true
toc_sticky: true

date: 2025-04-10
last_modified_at: 2025-04-10
---

## 들어가며

이번 시간에는 Texutre의 개념에 대해 알아보고, Mesh에 Texture를 입혀보는 방법을 알아본다.

지난 시간에 만든 직사각형에 이미지를 입혀보는 것이다. 직사각형의 정점에 UV 좌표를 부여하여 이미지를 붙여보자.

![TextureMapping](/assets/images/post_img/directx/TextureMapping.png)

---

## Texture

> 💡Texutre란?
>
> 3D 그래픽에서 폴리곤으로 제작된 오브젝트에 덧씌워 색상이나 질감 등을 표현하는 2D 이미지
>
> _출처: 네이버 게임 용어 사전_

&nbsp;

- Texture를 사용하는 이유?
    - 픽셀 단위로 색을 지정하지 않아도 자연스럽고 디테일한 표현이 가능하다.
    - 렌더링 파이프라인의 Rasterizer 단계에서 보간 처리를 통해 색상이 자연스럽게 입혀진다.

![PixelInterpolation](/assets/images/post_img/directx/PixelInterpolation.png)

---

### Lib 설정

Texutre Mapping에 필요한 DirectXTex 라이브러리를 추가해야 한다.
1. `Library/Lib`, `Library/Include` 폴더 생성
2. GitHub에서 DirectXTex 다운로드 및 압축 해제
3. `DirectXTex_Desktop_2019_Win10` 솔루션을 열고, Debug / Release 각각 빌드
4. `.lib` 파일 이름을 `DirectXTex_debug.lib`, `DirectXTex_release.lib`로 수정 후 `Library/Lib`에 복사
5. `DirectXTex.h`, `DirectXTex.inl`은 `Library/Include`로 복사
6. 각 프로젝트에 다음 설정 추가
    - Engine 프로젝트
        - EnginePch.h에 헤더 추가
            ```cpp
            #include <DirectXTex/DirectXTex.h>
            #include <DirectXTex/DirectXTex.inl>

            #ifdef _DEBUG
            #pragma comment(lib, "DirectXTex\\DirectXTex_debug.lib")
            #else
            #pragma comment(lib, "DirectXTex\\DirectXTex.lib")
            #endif
            ```
        - VC++ 디렉터리 → 포함 디렉터리 `$(SolutionDir)Library/Include/`
    - Client 프로젝트
        - VC++ 디렉터리 → 포함 디렉터리 `$(SolutionDir)Library/Include/` + 라이브러리 디렉터리 `$(SolutionDir)Library/Lib/` 추가

---

### Texture 클래스

- 멤버 변수
    - `_image`: 이미지 데이터를 로드해 임시 저장
    - `_tex2D`: 실제 GPU에 올려질 텍스처 자원
    - `_srvHeap`, `_srvHandle`: Shader Resource View 관련
    ```cpp
    private:
        ScratchImage			 		_image;
        ComPtr<ID3D12Resource>			_tex2D;

        ComPtr<ID3D12DescriptorHeap>	_srvHeap;
        D3D12_CPU_DESCRIPTOR_HANDLE		_srvHandle;
    ```

&nbsp;

- `Texture::Init()`
    ```cpp
    CreateTexture(path);  // 이미지 로딩 및 GPU 업로드
    CreateView();         // SRV 뷰 생성    
    ```
    - 텍스처 생성 (`Texutre::CreateTexture()`)
        - 설정 필요 사항
            - C++ 17 설정 (파일 경로 관련 기능): 속성 → C/C++ → 언어 → C++ 언어 표준 → C++ 7
            - `std::byte` 충돌 방지를 위해 `#define _HAS_STD_BYTE 0` (막지 않으면 `E0266 - byte가 모호합니다` 오류 발생)
        - 주요 기능
            1. 경로에 따라 이미지를 로딩한다. (포맷 자동 감지)
            2. `_tex2D` 리소스를 생성한다.
            3. 업로드 버퍼에 이미지를 복사한다.
            4. 커맨드 큐에 텍스처 로딩 명령을 추가한다. (커맨드 리스트 분리 필요 - Command Queue 확장 부분 참고)
    - View 생성 (`Texture::CreateView()`)
        - 주요 기능
            1. Descriptor Heap(SRV(Shader Resource View))을 생성한다
            2. Descriptopr의 위치를 가져와 `_srvHandle`에 저장한다.
            3. SRV 뷰 설명 구조체를 세팅한다.
            4. SRV를 생성한다.

---

### Command Queue 클래스

리소스 전용 커맨드 리스트를 추가한다. 굳이 리소스 전용을 사용하는 이유는 렌더링 명령과 리소스 초기화(업로드) 명령은 타이밍과 흐름이 다르기에 이를 분리해서 관리하기 위함이다.

일반적으로 Command List에서 렌더링은 `RenderBegin()` ~ `RenderEnd()` 사이에 이루어지는데, 이는 매 프레임 그리기 명령 전용으로 사용된다. 하지만 텍스처 리소스를 GPU에 올리는 작업은 한 번만 해도 되는 작업이다. 매 프레임마다 처리할 필요가 없기 때문에 렌더링용 Command List와 리소스용 Command List를 분리하는 것이다. 

&nbsp;

- 멤버 변수 

```cpp
ComPtr<ID3D12CommandAllocator>		_resCmdAlloc;
ComPtr<ID3D12GraphicsCommandList>	_resCmdList;
```

&nbsp;

- `CommandQueue::Init()` 수정

```cpp
device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&_resCmdAlloc));
device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, _resCmdAlloc.Get(), nullptr, IID_PPV_ARGS(&_resCmdList));
```

&nbsp;

- `CommandQueue::FlushResourceCommandQueue()`
    - 리소스 업로드 후 커맨드를 실행한다.

```cpp
_resCmdList->Close();
ID3D12CommandList* cmdListArr[] = { _resCmdList.Get() };
_cmdQueue->ExecuteCommandLists(_countof(cmdListArr), cmdListArr);

WaitSync();

_resCmdAlloc->Reset();
_resCmdList->Reset(_resCmdAlloc.Get(), nullptr);
```

---

### 레지스터 설정 

- `EnginePch.h`
    - 사용할 레지스터 설정을 위해 CBV와 SRV 레지스터 정의와 개수를 지정한다.
        ```cpp
        enum class CBV_REGISTER : uint8 { b0, b1, b2, b3, b4, END };

        enum class SRV_REGISTER : uint8
        {
            t0 = static_cast<uint8>(CBV_REGISTER::END),
            t1, t2, t3, t4, END
        };

        enum
        {
            SWAP_CHAIN_BUFFER_COUNT = 2,
            CBV_REGISTER_COUNT = CBV_REGISTER::END,
            SRV_REGISTER_COUNT = static_cast<uint8>(SRV_REGISTER::END) - CBV_REGISTER_COUNT,
            REGISTER_COUNT = CBV_REGISTER_COUNT + SRV_REGISTER_COUNT,
        };
        ```
    - Vertex 구조체에 UV 좌표를 추가한다.
        ```cpp
        struct Vertex
        {
            Vec3 pos;
            Vec4 color;
            Vec2 uv;
        };
        ```
        - `uv`: 텍스처의 좌표 (0.0 ~ 1.0 tkdl)
            - (0, 0): 이미지 왼쪽 위
            - (1, 1): 이미지 오른쪽 아래

---

### Shader 클래스

정점의 uv 좌표를 받아 픽셀 쉐이더에서 텍스처를 샘플링하여 출력하도록 Shader 파일을 수정해보자.

&nbsp;

- `default.hlsl`

```hlsl
cbuffer TEST_B0 : register(b0)
{
    float4 offset0;
};

cbuffer TEST_B1 : register(b1)
{
    float4 offset1;
};

Texture2D tex_0 : register(t0);
SamplerState sam_0 : register(s0);

struct VS_IN
{
    float3 pos : POSITION;
    float4 color : COLOR;
    float2 uv : TEXCOORD;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float4 color : COLOR;
    float2 uv : TEXCOORD;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);
    output.color = input.color;
    output.uv = input.uv;

    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
    float4 color = tex_0.Sample(sam_0, input.uv);
    return color;
}
```

&nbsp;

- `Shader::Init()` 수정
    - 쉐이더 파일을 수정했으니, Shader 클래스도 함께 수정해줘야 한다.
    - uv 좌표(TEXCOORD)를 받기 위해 입력 레이아웃에 TEXCOORD를 추가한다.

```cpp
D3D12_INPUT_ELEMENT_DESC desc[] =
{
    { "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0,  D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
    { "COLOR",    0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
    { "TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, 28, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
};
```

---

### Root Signature 클래스

- 멤버 변수 추가
    - `D3D12_STATIC_SAMPLER_DESC _samplerDesc;`
- `RootSignature::Init()` 수정
    - `CreateSamplerDesc()`와 `CreateRootSignature()`로 나눈다.
    - `CreateSamplerDesc()`
        ```cpp
        _samplerDesc = CD3DX12_STATIC_SAMPLER_DESC(0);
        ```
    - `CreateRootSignature()`
        ```cpp
        CD3DX12_DESCRIPTOR_RANGE ranges[] =
        {
            CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT, 0), // b0~b4
            CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, SRV_REGISTER_COUNT, 0), // t0~t4
        };

        CD3DX12_ROOT_PARAMETER param[1];
        param[0].InitAsDescriptorTable(_countof(ranges), ranges);

        D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param, 1, &_samplerDesc);
        sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT;

        ComPtr<ID3DBlob> blobSignature;
        ComPtr<ID3DBlob> blobError;
        ::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);

        DEVICE->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
        ```

---

### Texture Mapping

- Client 프로젝트의 `Game::Init()`
    - 정점 데이터에 uv 좌표를 추가한다.
        ```cpp
        vector<Vertex> vec(4);
        vec[0].pos = Vec3(-0.5f, 0.5f, 0.5f);  vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);  vec[0].uv = Vec2(0.f, 0.f);
        vec[1].pos = Vec3(0.5f, 0.5f, 0.5f);   vec[1].color = Vec4(0.f, 1.f, 0.f, 1.f);  vec[1].uv = Vec2(1.f, 0.f);
        vec[2].pos = Vec3(0.5f, -0.5f, 0.5f);  vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);  vec[2].uv = Vec2(1.f, 1.f);
        vec[3].pos = Vec3(-0.5f, -0.5f, 0.5f); vec[3].color = Vec4(0.f, 1.f, 0.f, 1.f);  vec[3].uv = Vec2(0.f, 1.f);
        ```
    - 텍스처 객체를 생성하고 초기화한다.
        ```cpp
        shared_ptr<Texture> texture = make_shared<Texture>();
        texture->Init(L"..\\Resources\\Texture\\dog.jpg");
        ```

&nbsp;

- Mesh 클래스 수정
    - 텍스처 포인터 멤버 변수와 Setter 함수를 추가한다.
        ```cpp
        shared_ptr<Texture> _tex;

        void SetTexture(shared_ptr<Texture> tex) { _tex = tex; }
        ```
    - Mesh::Render() 내부에 SRV 설정을 추가한다.
        ```cpp
        GEngine->GetTableDescHeap()->SetSRV(_tex->GetCpuHandle(), SRV_REGISTER::t0);
        ```

&nbsp;

- TableDescriptorHeap 클래스 수정
    - `SetSrv()`를 구현한다. 
        ```cpp
        D3D12_CPU_DESCRIPTOR_HANDLE destHandle = GetCPUHandle(reg);

        uint32 destRange = 1;
        uint32 srcRange = 1;

        DEVICE->CopyDescriptors(
            1, &destHandle, &destRange,
            1, &srcHandle, &srcRange,
            D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV
        );
        ```

&nbsp;

그럼 넣어준 흰둥이 사진이 잘 출력된다!

![dog](/assets/images/post_img/directx/TextureMappingResult.png)

---

[Texture 커밋](https://github.com/chaeeun-dev/DirectX12/commit/0de44414b4af0dc54e4a367637f16a699aca4f0f)

---

## 마치며

- 이번 시간에 배운 것
    1. Shader에서 텍스처와 uv 좌표 처리하기
    2. Root Signature에서 샘플러 설정하기
    3. Vertex에서 uv 값을 설정하여 텍스처 좌표 지정 후 텍스처를 로드하고, Mesh 클래스에서 텍스처를 연결하기

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*