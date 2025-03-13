---
title: "[DirectX 12] #2-4. Constant Buffer"
excerpt: "Root Signature와 Constant Buffer 이해하기"

categories:
  - DirectX12
tags:
  - [DirectX, RootSignature, ConstantBuffer]

permalink: /DirectX12/constant-buffer/

toc: true
toc_sticky: true

date: 2025-03-13
last_modified_at: 2025-03-13
---

## 규칙

쉐이더 파일은 정해진 규칙에 따라 작성해야 하며, 렌더링 파이프라인의 각 단계에서 특정 역할을 한다. 특히 Input Assembler 단계에서는 한정된 정점 정보만 전달할 수 있는데, 추가적인 데이터를 전달하려면 Constant Buffer와 같은 다른 방법을 사용해야 한다. 또, 데이터를 무작정 GPU로 보낼 수 없고, RootSignature 서명 단계를 거쳐야 한다. 

---

## CPU vs GPU

CPU와 GPU는 성능을 최적화하기 위해 계층적인 메모리 구조를 가진다.

![Image](https://github.com/user-attachments/assets/28901293-1031-4896-b351-d746c5c42f40)

1. CPU 메모리 계층 구조
    - Register → Cache → RAM 순으로 데이터를 접근한다.
    - 가까울수록 속도가 빠르지만, 용량이 작고 비용이 비싸다.
    - RAM은 크지만 접근 속도가 느려, 항상 Register나 Cache를 먼저 확인한다.
2. GPU 메모리 계층 구조
    - GPU도 Register → Cache → RAM 순으로 데이터를 접근한다.
    - 루트 시그니처를 통해 Register와 Cache를 어떻게 사용할지 결정해야 한다. 

---

## Shader

저번 삼각형 그리기 수업에서는 아무 데이터도 전달하지 않은 빈 Root Signature를 사용했다. 이번에는 Constant Buffer를 활용하여 쉐이더에 데이터를 전달하는 방식을 알아본다.

&nbsp;

- `default.hlsli` 쉐이더 파일에 Constant Buffer 추가 
    - TEST_B0(b0 레지스터)와 TEST_B1(b1 레지스터)를 추가한다.
    -  Constant Buffer를 통해 `offset0`과 `offset1` 값을 전달하고, 이를 정점 위치와 색상 값에 저장한다.

```cpp
cbuffer TEST_B0 : register(b0)
{
    float4 offset0;
};

cbuffer TEST_B1 : register(b1)
{
    float4 offset1;
};


VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);
    output.pos += offset0;
    output.color = input.color;
    output.color += offset1;

    return output;
}

```

---

## Root Signature 예제

1. 빈 Root Signature
    
    ![Image](https://github.com/user-attachments/assets/9b0d69e1-4884-4b28-a823-cd1564a0de47)
    
    - 데이터를 전달하지 않는 가장 단순한 형태의 Root Signature이다.
    - 이 Root Signature는 데이터를 직접 전달하지 않고, Input Assenbler와 최소한의 쉐이더만 사용하여 렌더링하는 경우에 사용된다. (저번 삼각형 그리기의 Root Signature)
2. Root Constant
    
    ![Image](https://github.com/user-attachments/assets/134e8336-632e-4360-875f-650367e2efe4)

    - Root Constant를 이용하여 데이터를 전달하는 방식이다.
    - API Bind Slot: 데이터가 바인딩 될 슬롯을 정의한다. 
    - HLSL Bind Slot: 쉐이더에서 사용할 레지스터를 정의한다. (`b0`, `b2` 등)
    - Root Constant: 상수 값을 전달할 때 사용한다. 
    - 참고. 루트 시그니처는 계약서와 같으며, 실제 데이터를 포함하지 않는다.
3. Root Constant Buffer View
    
    ![Image](https://github.com/user-attachments/assets/f1d1fd99-26cc-4afe-bb37-75363fc8d70a)

    - Constant Buffer를 Root Signature에 추가하는 방식이다. 
    - Root Descriptor(View)를 통해 `rootCBV`를 추가하여, Constant Buffer를 가리키도록 설정한다.
    - Root Descriptor는 포인터처럼 다른 리소스를 가리키는 개념이다. 
4. Descriptor Table 
    
    ![Image](https://github.com/user-attachments/assets/357760fd-1995-43cf-bf92-385a5a69f43e)

    - Descriptor Table을 이용해 다양한 리소스를 관리하는 방식이다.
    - Descriptor Table은 특정 슬롯에 여러 개의 리소스를 한 번에 넣을 수 있는 방식이다.
    - Root Signature를 설정한 후 어떤 데이터를 사용할지 정의해야 한다.
    - Root Table의 크기는 제한적(4byte x 64개)이므로, 포인터 방식으로 데이터를 참조한다.

---

## Constant Buffer View

이번 수업에서는 Constant Buffer View(CBV)를 이용해 데이터를 쉐이더로 전달하는 방식을 다룬다. 

정리하면, 쉐이더에 일감을 맡기기 위해 Root Signature로 계약을 맺었으니 GPU가 실질적으로 사용할 데이터를 Constant Buffer에 저장하고, Constant Buffer View로 Root Signature에서 예약한 레지스터(`b0`, `b1`)에 Constant Buffer를 연결하는 것이다.

&nbsp;

- 기존의 Root Signature 수정 (`RootSignature::Init()`)
    - 기존의 Root Signature는 아무 작업도 하지 않았지만, 이번에는 CBV를 추가하여 데이터를 전달하도록 수정한다.
    - 수정 내용
        - Root CBV 2개 추가 → Root Signature 예제의 Root Constant Buffer View 방식
        - `param[0].InitAsConstantBufferView(0);` → 0번 슬롯에 rootCBV 추가 (b0 레지스터에 매핑)
        - `param[1].InitAsConstantBufferView(1);` → 1번 슬롯에 rootCBV 추가 (b1 레지스터에 매핑)
    - 참고. `D3D_SHADER_VISIBILITY_ALL` → 파이프라인을 거치면서 데이터가 소멸되지 않도록 한다.
    - 이 과정은 땅을 계약하는 과정이라고 할 수 있다. 아직 집은 짓지 않은 상태이다.
    - 아직 데이터를 직접 활용하지 않으며, Mesh Rendering에서 실질적인 작업을 추가해야 한다.

```cpp
CD3DX12_ROOT_PARAMETER param[2];
param[0].InitAsConstantBufferView(0); // 0번 -> b0 -> CBV 
param[1].InitAsConstantBufferView(1); // 1번 -> b1 -> CBV

D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(2, param);
sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

ComPtr<ID3DBlob> blobSignature;
ComPtr<ID3DBlob> blobError;
::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
device->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
```

&nbsp;

- Root Signature 서명 활용 (`CommandQueue::RenderBegin()`)
    - Root Signature를 적용하기 위해 Command List에 설정을 한다.
    - 이 코드를 추가하면 Root Signature가 설정되고, 쉐이더에 데이터를 받을 준비가 완료된다. 

```cpp
_cmdList->SetGraphicsRootSignature(ROOT_SIGNATURE.Get());
```

&nbsp;

- Mesh Redering에서 CBV 활용 (`Mesh::Rendering()`)
    - Command Queue에 Root Signature를 추가했으므로, 이제 Command List를 통해 CBV를 설정하는 코드를 작성해야 한다. 
    - 현재의 문제점
        - `CMD_LIST->SetGraphicsRootConstantBufferView(0, 데이터);` 이렇게 작성하고 싶지만 아직 실행할 수 없다.
        - 그 이유는 CBV를 위한 Constant Buffer 리소스를 생성하고, 데이터를 설정하는 과정이 먼저 필요하기 때문이다. 

&nbsp;

- `Mesh::Init()`
    - Constant Buffer 리소스 생성
        - Root Signature에서 CBV로 `b0`, `b2`를 설정했지만, 실제 데이터를 담을 Constant Buffer 리소스를 아직 생성하지 않았다. `CreateCommittedResource()`를 이용해 리소스를 생성한다.
        - `D3D12_HEAP_TYPE_UPLOAD` → 업로드 타입으로 데이터를 CPU에서 GPU로 전달한다.
        - `D3D12_RESOURCE_DESC::Buffer(sizeof(ConstantBuffer))` → Constant Buffer 크기만큼의 버퍼를 생성한다. 
        ```cpp
        DEVICE->CreateCommittedResource(
            &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
            D3D12_HEAP_FLAG_NONE,
            &CD3DX12_RESOURCE_DESC::Buffer(sizeof(ConstantBuffer)),
            D3D12_RESOURCE_STATE_GENERIC_READ,
            nullptr,
            IID_PPV_ARGS(&_constantBuffer));
        ```
    - Constant Buffer에 데이터 설정
        - 생성한 Constant Buffer에 데이터를 복사하려면 Map과 Unmap을 사용해야 한다.
        - `Map()` → 버퍼를 CPU에서 접근하도록 설정
        - `memcpy()` → 데이터를 복사하여 GPU에 전달
        - `Unmap()` → 다시 GPU가 사용할 수 있도록 설정
        - 💡여기서 중요한 점은 Device를 사용하기 때문에 즉시 실행된다는 것이다. 반면, Command List를 사용하는 작업은 예약만 하고 나중에 실행된다.
        ```cpp
        void* mappedData;
        _constantBuffer->Map(0, nullptr, reinterpret_cast<void**>(&mappedData));
        memcpy(mappedData, &constantBufferData, sizeof(ConstantBuffer));
        _constantBuffer->Unmap(0, nullptr);
        ```

&nbsp;

- 💡 실행 시점의 차이(중요)
    - Command Queue는 작업을 예약하고, 나중에 한 번에 실행한다.
    - Device 작업은 즉시 실행된다.
    - 실행 순서 문제 예시
        1. 삼각형 A를 렌더링하기 위해 Buffer에 값 1을 설정한다.
        2. CBV 주소를 예약한다. (Command List 사용)
        3. 삼각형 B를 렌덜이하려고 Buffer에 값 2를 설정한다.
        4. CBV 주소를 예약한다.
        5. Command List 실행 시, 두 개의 삼각형이 모두 2를 사용하게 된다. (덮어쓰기 문제)
    - 해결 방법
        - 버퍼를 여러 개 생성하여 덮어쓰기 문제를 방지한다. 

![Image](https://github.com/user-attachments/assets/32345fd7-f2f8-48c2-a70b-92345a91096a) 

&nbsp;

- 흐름 정리
1. Root Sigature 설정 (쉐이더와 계약서 작성) → 쉐이더에게 b0, b1를 사용하겠다고 계약을 맺는다.
2. Constant Buffer 생성 (데이터 저장소 마련) → 창고(b0, b1)에 데이터를 채운다.
3. Constant Buffer View 연결 (데이터 전달 설정) → b0에 넣을 데이터, b1에 넣을 데이터를 설정한다.
4. 쉐이더 값 사용 → 이제 쉐이더가 데이터를 가져가서 연산을 수행할 수 있다!

---

## Constant Buffer

Constant Buffer의 역할
- Root Signature에서 선언한 b0, b1에 데이터를 보내기 위한 버퍼이다.
- CPU에서 데이터를 작성 → GPU가 사용하는 구조이다.
- 실행 타이밍이 다르므로 데이터 덮어쓰기 문제가 발생할 수 있다. → 버퍼를 여러개 사용하여 문제를 해결한다.

&nbsp;

- 사용 변수

```cpp
ComPtr<ID3D12Resource> _cbvBuffer;   // 실제 Constant Buffer 리소스
BYTE* _mappedBuffer = nullptr;       // CPU가 데이터를 복사하는 버퍼
uint32 _elementSize = 0;             // 한 개 요소 크기 (256의 배수)
uint32 _elementCount = 0;            // 요소 개수 (한 프레임에 사용할 개수)
uint32 _currentIndex = 0;            // 현재 사용 중인 인덱스 (프레임 끝나면 0으로 초기화)
```

&nbsp;

- 초기화 (`ConstantBuffer::Init()`)
    - 버퍼 크기를 256의 배수로 맞춰야 하므로 `(size + 255) & ~255` 연산을 사용한다.
    - `_elementSize * _elementCount` 크기의 버퍼를 생성한다.

```cpp
_elementSize = (size + 255) & ~255;
_elementCount = count;
CreateBuffer();
```

&nbsp;

- 버퍼 생성 (`ConstantBuffer::CreateBuffer()`)
    - `D3D12_HEAP_TYPE_UPLOAD`을 사용하여 CPU에서 직접 접근 가능하도록 설정한다.
    - `Map()`을 호출하여 `_mappedBuffer`에 연결한다.
    - `Unmap()`을 하지 않는 이유는 GPU가 사용할 때 데이터를 변경하면 안 되기 때문이다.

```cpp
uint32 bufferSize = _elementSize * _elementCount;
D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD);
D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Buffer(bufferSize);

DEVICE->CreateCommittedResource(
    &heapProperty,
    D3D12_HEAP_FLAG_NONE,
    &desc,
    D3D12_RESOURCE_STATE_GENERIC_READ,
    nullptr,
    IID_PPV_ARGS(&_cbvBuffer));

_cbvBuffer->Map(0, nullptr, reinterpret_cast<void**>(&_mappedBuffer));
```

&nbsp;

- 데이터 넣기 (`ConstantBuffer::PushData()`)
    1. 디버그 체크
    2. CPU 메모리에 데이터를 복사한다.
    3. GPU 주소를 가져와서 CBV를 설정한다. 
    - 실행 타이밍 주의하기 → `memcpy()`는 즉시 실행, `SetGraphicsConstantBufferView()`는 Command List에 기록되어 나중에 실행된다. 

```cpp
assert(_currentIndex < _elementSize);

::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

D3D12_GPU_VIRTUAL_ADDRESS address = GetGpuVirtualAddress(_currentIndex);
CMD_LIST->SetGraphicsRootConstantBufferView(rootParamIndex, address);
_currentIndex++;
```

&nbsp;

- GPU 가상 주소 가져오기 (`ConstantBuffer::GetGpuVirtualAddress()`)
    - GPU에서 Constant Buffer의 시작 주소를 가져오고, 해당 인덱스만큼 offset을 더해 사용한다. 

```cpp
D3D12_GPU_VIRTUAL_ADDRESS objCBAddress = _cbvBuffer->GetGPUVirtualAddress();
objCBAddress += index * _elementSize;
return objCBAddress;
```

&nbsp;

- 버퍼 생성 및 적용하기
    - `Engine`에서 버퍼 생성
        - `Engine` 클래스에 `shared_ptr<ConstantBuffer>` 멤버 변수를 추가한다.
        - `Engine::Init()`에서 Constant Buffer를 생성한다.
        ```cpp
        _cb = make_shared<ConstantBuffer>();
        _cb->Init(sizeof(Transform), 256);
        ```
    - `CommandQueue`에서 초기화
        - `RenderBegin()`에서 `_currentIndex`를 초기화한다. (0으로 초기화)
        ```cpp
        GEngine->GetCB()->Clear();
        ```
    - `Mesh`에서 Transform 적용
        - `Mesh` 클래스에 `Transform _transform = {};` 멤버 변수를 추가한다.
        - `SetTransform()` 함수를 추가한다.
        ```cpp
        void SetTransform(const Transform& t) { _transform = t; }
        ```
        - `Mesh::Render()`에서 CBV에 Transform 데이터를 넣는다. (b0, b1에 각각 Transform 데이터를 넣는다.)
        ```cpp
        GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
        GEngine->GetCB()->PushData(1, &_transform, sizeof(_transform)); 
        ```

---

## 삼각형 실습

- `Game::Update()`
    - Constant Buffer가 잘 적용되는지 확인해본다.

```cpp
{
    Transform t;
    t.offset = Vec4(0.75f, 0.f, 0.f, 0.f);
    mesh->SetTransform(t);
    mesh->Render();
}

{
    Transform t;
    t.offset = Vec4(0.f, 0.75f, 0.f, 0.f);
    mesh->SetTransform(t);
    mesh->Render();
}
```

![Image](https://github.com/user-attachments/assets/2e775b1c-74f3-4cf4-a99b-c5f01542564f)

- 결과 
    - 첫 번째 삼각형 → X 좌표 0.75f 증가, R 값 0.75
    - 두 번째 삼각형 → Y 좌표 0.75 증가, G 값 0.75


---

## 마치며 

- 이번 수업에서 가장 중요한 것
    - 즉시 실행되는 일과 나중에 실행되는 일로 인해 발생하는 실행 타이밍 문제를 해결하기 위해 버퍼를 여러개로 나누어 관리한다.

~~다렉 기초인데 너무 어렵다......... 머리가 터질 것 같다.... 그래도 파이팅..~~

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*
*[MS 홈페이지](https://learn.microsoft.com/en-us/windows/win32/direct3d12/example-root-signatures)*