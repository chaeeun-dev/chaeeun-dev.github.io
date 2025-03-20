---
title: "[DirectX 12] #2-3. 삼각형 띄우기"
excerpt: "여러 장치를 이용해 삼각형 띄우기"

categories:
  - DirectX12
tags:
  - [DirectX, Triangle]

permalink: /DirectX12/draw-triangle/

toc: true
toc_sticky: true

date: 2025-03-10
last_modified_at: 2025-03-10
---

## 들어가며

지난 수업에서 각 장치를 클래스로 나누어 관리했는데, 일부 책이나 강의에서는 이를 한 번에 관리하는 방식도 사용한다. 클래스로 따로 관리하면 컴포넌트 구조 및 종속 관계(예. Swap Cahin과 Descriptor Heap)를 이해하는 데 도움이 된다.

이번 수업에서는 저번 수업의 코드를 수정한 후 각 장치와 Mesh, Shader 등을 이용해 삼각형을 화면에 그려본다. 

---

## 코드 정리하기

- Descriptor Heap 관련 변수 이동

공통된 변수 관리를 위해 `DescriptorHeap.h`에 있던 변수를 `SwapChain.h`로 이동한다. 그리고 필요 없는 `rtvHeapSize` 변수는 삭제한다.

```cpp
ComPtr<ID3D12DescriptorHeap>	_rtvHeap;
D3D12_CPU_DESCRIPTOR_HANDLE		_rtvHandle[SWAP_CHAIN_BUFFER_COUNT];
```

&nbsp;

- 변수명 수정 

기존 `SwapChain.h`의 기존 `_renderTargets` 변수를 `_rtvBuffer`로 변경한다. (전체 솔루션에서 변수 이름 변경하기: `Ctrl + R, R`)

&nbsp;

- `SwapChain::CreateRTV()` 

기존 `Descriptor::Init()` 함수를 SwapChain의 `CreateRTV(ComPtr<ID3D12Device> device)`로 이동 후 수정한다. 이제 `SwapChain::Init()`에서 `CreateRTV()`를 호출하여 초기화한다. 이 과정에서 `Init()` 함수에 `device` 매개변수가 없으므로 추가한다.

```cpp
SwapChain::Init(const WindowInfo& info, ComPtr<ID3D12Device> device, 
ComPtr<IDXGIFactory> dxgi, ComPtr<ID3D12CommandQueue> cmdQueue);
```

&nbsp;

- `SwapChain::Init()` 세분화

기존의 `Init()`을 두 개의 함수로 분리하여 역할을 명확하게 나눈다.
    - `CreateSwapChain()`
    - `CreateRTV()`

&nbsp;

- `Descriptor.h` & `Descriptor.cpp` 파일 삭제하기

파일을 삭제한 후, 빌드해서 발생하는 include 오류와 다른 오류를 수정한다.

&nbsp;

- `SwapChain.h` 수정
    - `GetCurrentBackBufferIndex()`는 필요 없으므로 삭제한다.
    - `_rtvHandle[_backBuffer]`를 반환하는 `D3D12_CPU_DESCRIPTOR_HANDLE GetBackRTV()`를 추가한다.
    - `GetCurrentBackBufferResource()` 함수명을 `GetBackRTVBuffer()`로 변경한다.
    - `CommandQueue::RenderBegin()`에서 `_descHeap->GetBackBufferView()`를 `_swapChain->GetBackRTV()`로 수정한다.

&nbsp;

- `Engine`에서 장치 접근을 위한 Get 함수 추가

각 장치들을 `Engine`에서 쉽게 접근할 수 있도록 `Get` 함수를 각각 추가한다. 

```cpp
public:
	shared_ptr<class Device> GetDevice() { return _device; }
	shared_ptr<class CommandQueue> GetCommandQueue() { return _cmdQueue; }
	shared_ptr<class SwapChain> GetSwapChain() { return _swapChain; }
```

&nbsp;

- `GetComandList()` 추가

`CommandQueue`와 마찬가지로 `CommandList`도 자주 사용되므로, `CommandQueue.h`에 `GetCommandList()`를 추가한다.

```cpp
ComPtr<ID3D12CommandQueue> GetCmdQueue() { return _cmdQueue; }
ComPtr<ID3D12GraphicsCommandList> GetCmdList() { return _cmdList; }
```

&nbsp;

- `EnginePch.h`에 자주 사용할 매크로 추가

예를 들어 `device` 매개변수를 전달하는 대신, 매크로를 사용해 쉽게 접근할 수 있도록 설정한다.

```cpp
#define DEVICE GEngine->GetDevice()->GetDevice()
#define CMD_LIST GEngine->GetCmdQueue()->GetCmdList()
```

&nbsp;

- `Engine.h`에 각 장치 헤더 포함하기

자주 포함해야 하는 헤더들을 `Engine.h`에 추가해 `Engine.h`만 include해도 다른 장치들의 헤더가 포함되도록 한다. 이렇게 하면 `Engine.h`만 include하면 필요한 장치들이 포함되므로, 기존에 사용되던 전방 선언도 삭제할 수 있다. 

```cpp
#include "Device.h"
#include "CommandQueue.h"
#include "SwapChain.h"
```

---

[코드 정리 커밋](https://github.com/chaeeun-dev/DirectX12/commit/1a9546d8ff7ca800b1053bbb56efc8a3b7862f59)

---

## 삼각형 띄우기

RootSignature와 Mesh, Shader의 개념을 이해하고 코드로 구현한 뒤, 삼각형을 그려보자.

---

### Root Signature

- Root Signature는 계약서를 결재하는 역할이라고 할 수 있다. 
- CPU는 RAM에 있는 데이터를 계속 꺼내면서 사용하지만, GPU는 RAM과 거리가 멀어 직접 사용할 수 없다. 그래서 CPU가 GPU에게 외주를 맡길 때 데이터를 어떤 형식으로 전달할지를 정해야 한다. 
- 예를 들어, 한국(CPU)에서 베트남(GPU)에게 외주를 맡길 때 원석(데이터)를 보내서 가공(일감 처리)해달라고 하는 것과 같다. 여기서 Root Signature는 서명처럼 어떤 정책을 적용할지와 어떤 땅(GPU자원)을 임대할지를 정하는 계약서 역할을 한다.

&nbsp;

- 사용 변수

```cpp
ComPtr<ID3D12RootSignature>	_signature;
```

&nbsp;

- 초기화 (`RootSignature::Init()`)
  1. 기본 상태에서 서명(계약)을 한다. (땅만 둘러봄)
  2. `Flag` 값을 설정해 입력 조립기 단계에서 사용할 수 있도록 설정한다.
  3. `D3D12SerializeRootSignature()`와 `CreateRootSignature()`를 호출해 RootSignature를 생성한다.
  4. 최종적으로 `_signature`에 정보를 저장한다.

```cpp
D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(D3D12_DEFAULT);
sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

ComPtr<ID3DBlob> blobSignature;
ComPtr<ID3DBlob> blobError;
::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
device->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
```

&nbsp;

- RootSignature 가져오기 (`RootSignature::GetSignature()`)
  - 초기화된 `_signature` 정보를 반환한다.

```cpp
ComPtr<ID3D12RootSignature> GetSignature() { return _signature; }
```

&nbsp;

- `Engine` 클래스에 `RootSignature` 추가하기
  1. `Engine.h`의 `Engine` 클래스에 `shared_ptr<RootSignature> _rootSignature` 멤버 변수를 추가한다.
  2. `Engine::Init()`에서 `_rootSignature`를 초기화한다.

```cpp
_rootSignature->Init(_device->GetDevice());
```

---

### Mesh

- Mesh는 정점(vertex)으로 이루어진 물체를 의미한다. 
- Resource 폴더에 `Mesh` 클래스를 생성해 관리한다.

&nbsp;

- Vertex 구조체 추가
  - `EnginePch.h`에 정점의 위치와 색상을 저장할 구조체를 만든다.
    - `Vec3 pos` → 위치 (x, y, z)
    - `Vec4 color` → 색상 (R, G, B, A)
    - 참고. Vec3는 float형이 3개, Vec4는 float형이 4개 있는 자료형이다.

```cpp
struct Vertex
{
	Vec3 pos;
	Vec4 color;
};
```

&nbsp;

- 사용 변수

```cpp
ComPtr<ID3D12Resource>		_vertexBuffer;
D3D12_VERTEX_BUFFER_VIEW	_vertexBufferView = {};
uint32 _vertexCount = 0;
```

&nbsp;

- 초기화 (`Mesh::Init(vector<Vertex>& vec))`)
  1. vec의 정보로 정점 개수를 저장한다. (`_vertexCount = vec.size()`)
  2. GPU에 사용할 버퍼 정보(`heapProperty` - 어떤 용도로 사용할지, `desc` - 얼만큼 할당 받을지)를 설정한다.
  3. GPU 내부에 `_vertexBuffer` 공간을 할당한다. (RAM이 아니라 GPU 안에 있는 공간이다. 이 공간에는 바로 복사할 수 없고 다음 단계를 거쳐 복사해야 한다.)
    - Map() → 데이터를 복사할 수 있도록 연결
    - memcpy() → 데이터를 복사
    - Unmap() → 연결 해제
  4. 버퍼 정보를 담은 View(`_vertexBufferView`)를 설정한다. (실질적으로 활용하는 데이터는 Buffer가 아닌 View이므로)

```cpp
_vertexCount = static_cast<uint32>(vec.size());
uint32 bufferSize = _vertexCount * sizeof(Vertex);

D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

DEVICE->CreateCommittedResource(
	&heapProperty,
	D3D12_HEAP_FLAG_NONE,
	&desc,
	D3D12_RESOURCE_STATE_GENERIC_READ,
	nullptr,
	IID_PPV_ARGS(&_vertexBuffer));

// Copy the triangle data to the vertex buffer.
void* vertexDataBuffer = nullptr;
CD3DX12_RANGE readRange(0, 0); // CPU에서 읽지 않을 범위 설정
_vertexBuffer->Map(0, &readRange, &vertexDataBuffer);
::memcpy(vertexDataBuffer, &vec[0], bufferSize);
_vertexBuffer->Unmap(0, nullptr);

// Initialize the vertex buffer view.
_vertexBufferView.BufferLocation = _vertexBuffer->GetGPUVirtualAddress();
_vertexBufferView.StrideInBytes = sizeof(Vertex); // 정점 1개 크기
_vertexBufferView.SizeInBytes = bufferSize; // 버퍼 크기
```

&nbsp;

- 그리기 (`Mesh::Render()`)
  - `CommandList`를 이용해 그림을 그린다.
  - `CommandQueue::RenderBegin()`과 `CommandQueue::RenderEnd()` 사이에 실행한다.
  1. 도형을 삼각형으로 설정한다.
  2. `IASetVertexBuffers()`를 호출해 슬롯 번호를 선택하고, `Init()`에서 초기화한 View를 사용한다. 
  3. `DrawInstanced()`를 호출해 그린다. 

```cpp
CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView); // Slot: (0~15)
CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
```

---

### Shader

- Shader는 일감 기술서라고 할 수 있다. 외주 인력(GPU)이 무엇을 해야 할지 기술하는 문서라고 보면 된다.

&nbsp;

- 쉐이더 파일 생성
  1. `Shader` 폴더에 `default.hlsli` 파일을 생성한다. 
  2. 이 파일은 C++이 아니라 **HLSL(High-Level Shader Language)**로 작성해야 한다.
  3. `VS_Main`은 Vertex Shader 단게에서 해야 할 일을 정의하고, `PS_Main`은 Pixel Shader 단계를 정의하는 역할을 한다. 

```hlsl
struct VS_IN
{
    float3 pos : POSITION;
    float4 color : COLOR;
};

struct VS_OUT
{
    float4 pos : SV_Position;
    float4 color : COLOR;
};

VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);  // float3인 pos에 1.f를 추가하여 float4로 변환
    output.color = input.color;  // 그대로 전달

    return output;
}

float4 PS_Main(VS_OUT input) : SV_Target
{
    return input.color; // 색상 그대로 반환
}
```

&nbsp;

- 사용 변수
  - 쉐이더 컴파일 결과를 저장하는 변수들이다.

```cpp
ComPtr<ID3DBlob>					_vsBlob;
ComPtr<ID3DBlob>					_psBlob;
ComPtr<ID3DBlob>					_errBlob;
ComPtr<ID3D12PipelineState>			_pipelineState;
D3D12_GRAPHICS_PIPELINE_STATE_DESC  _pipelineDesc = {};
```

&nbsp;

- 초기화 (`Shader::Init()`)
  1. 매개 변수로 받은 경로에 있는 쉐이더 파일을 읽어서 Vertex Shader와 Pixel Shader를 생성한다.
  2. `_vsBlob`과 `psBlob` 변수에 쉐이더 관련 리소스를 저장한다.
  3. 그래픽스 파이프라인을 설정하고 `CreateGraphicsPipelineState()`를 호출하여 생성한다.

```cpp
CreateVertexShader(path, "VS_Main", "vs_5_0");
CreatePixelShader(path, "PS_Main", "ps_5_0");

D3D12_INPUT_ELEMENT_DESC desc[] =
{
    { "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
    { "COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
};

_pipelineDesc.InputLayout = { desc, _countof(desc) };
_pipelineDesc.pRootSignature = ROOT_SIGNATURE.Get();

_pipelineDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
_pipelineDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
_pipelineDesc.DepthStencilState.DepthEnable = FALSE;
_pipelineDesc.DepthStencilState.StencilEnable = FALSE;
_pipelineDesc.SampleMask = UINT_MAX;
_pipelineDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
_pipelineDesc.NumRenderTargets = 1;
_pipelineDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
_pipelineDesc.SampleDesc.Count = 1;

DEVICE->CreateGraphicsPipelineState(&_pipelineDesc, IID_PPV_ARGS(&_pipelineState));
```

&nbsp;

- 업데이트 (`Shader::Update()`)
  - 쉐이더 파이프라인을 설정한다. 

```cpp
CMD_LIST->SetPipelineState(_pipelineState.Get());
```

---

### 삼각형 그리기

삼각형을 그리기 위한 **Root Signature**, **Mesh**, **Shader** 작업이 완료됐다. 이제 삼각형을 그려보자.

&nbsp;

- 초기화 (`Game::Init()`)
  1. `GEngine`을 초기화한다.
  2. 그릴 삼각형의 정점(Vertex) 정보를 저장한다.
  3. 쉐이더를 초기화한다.
  4. `GEngine`을 동기화한다.

```cpp
GEngine->Init(info);

vector<Vertex> vec(3);
vec[0].pos = Vec3(0.f, 0.5f, 0.5f);
vec[0].color = Vec4(1.f, 0.f, 0.f, 1.f);
vec[1].pos = Vec3(0.5f, -0.5f, 0.5f);
vec[1].color = Vec4(0.f, 1.f, 0.f, 1.f);
vec[2].pos = Vec3(-0.5f, -0.5f, 0.5f);
vec[2].color = Vec4(0.f, 0.f, 1.f, 1.f);
mesh->Init(vec);

shader->Init(L"..\\Resources\\Shader\\default.hlsli");

GEngine->GetCmdQueue()->WaitSync();
```

&nbsp;

- 업데이트 (`Game::Update()`)
  - `RenderBegin()`과 `RenderEnd()` 사이에서 Shader와 Mesh를 각각 업데이트, 렌더링 한다.

```cpp
GEngine->RenderBegin();

shader->Update();
mesh->Render();

GEngine->RenderEnd();
```

&nbsp;

- 결과 
  - 정점을 3개만 지정했는데도 파이프라인을 거치면서 내부 정점들이 보간되어 채워진다. 색상 역시 보간되어 자연스럽게 변한다.
  - 삼각형이 출력되지 않는 문제 해결
    - 삼각형이 그려지지 않고 파란 화면만 떠서 30분 넘게 헤맸는데, 원인은 Viewport 설정 때문이었다. `_viewport = { 0, 0, static_cast<FLOAT>(info.height), 0.0f, 1.0f }; `로 되어있던 코드를 `_viewport = { 0, 0, static_cast<FLOAT>(info.width), static_cast<FLOAT>(info.height), 0.0f, 1.0f };` 이렇게 수정하니 정상적으로 출력됐다. 
    - 뷰포트는 그림을 그릴 캔버스의 크기를 설정하는 과정이다. 앞서 설정한 코드의 3, 4 번째 인자가 캔버스의 너비와 높이를 설정하는 건데 인자 값을 잘못 넣어서 그림을 그릴 공간이 없던 것이다. 

![Image](https://github.com/user-attachments/assets/4397c485-d250-4cbf-a4cd-659192e2b2fb)

---

[삼각형 띄우기 실습 커밋](https://github.com/chaeeun-dev/DirectX12/commit/db1c3c0f26ee67be773da3dbdb0d8ef16119b8b6)

---

## 마치며

점점 DX12 코드가 방대해지다보니 깜빡하고 초기화 하지 못한 코드가 있거나, 자동완성으로 오타를 내는 경우가 많다. 그래서 수업 듣고 코드 넣고, 빌드하면서 오류 고치는 데 한세월이 걸린다. 그래도 계속 겪게 될 일들이니 익숙해지자 ㅎ_ㅎ 

그래도 내 손으로 직접 오류 찾으니까 뿌듯한 기분이 든다.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*