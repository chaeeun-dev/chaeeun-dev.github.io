---
title: "[DirectX 12] #2-5. Root Signature"
excerpt: "Root Table로 Constant Buffer 관리하기"

categories:
  - DirectX12
tags:
  - [DirectX]

permalink: /DirectX12/root-signature/

toc: true
toc_sticky: true

date: 2025-03-21
last_modified_at: 2025-03-21
---

## 들어가며

이번에는 Root Signature 방식을 바꿔볼 것이다. 이전 수업에서는 Root Signature에 Root CBV Descriptor 자체를 넣어 사용하는 방식(Adding a Root Constant Buffer View)을 사용했다. 이번에는 Root Table을 사용하여 Constant Buffer를 관리하는 방식(Binding Descriptor Tables)을 사용한다.

[#2-4. Constant Buffer](https://chaeeun-dev.github.io/DirectX12/constant-buffer/#constant-buffer) 포스트의 Root Signature 예제 참고

- 왜 새로운 방식을 사용하는가?
    - Root Signature의 Root 파라미터 크기는 최대 4바이트 x 64개로 제한적이다.
    - CBV가 많아지면 Root Signature에 직접 넣는 방식이 비효율적이다.
    - Descriptor Table을 사용하면 더 많은 CBV를 관리할 수 있다.

---

## Root Table


이제 Root Signature에 직접 CBV를 포함하는 대신, Root Table을 이용해 여러 개의 CBV를 관리하는 방식을 사용한다.

![Image](https://github.com/user-attachments/assets/cf04cbba-85e2-4162-8c6d-12297574a6cc)

- Root Table이란?
    - Root Table은 Descriptor Heap을 가리키는 포인터 역할을 한다.
    - CBV를 Root Signature에 직접 넣는 대신, Root Table이 Descriptor Heap을 참조하도록 설정한다. 
    - Descriptor Heap 내부에 어려 개의 CBV를 저장해 관리할 수 있다.
    - `SetDescriptorHeaps()`와 `SetGraphicsRootDescriptorTable()`을 사용해 데이터를 GPU에 전달한다.
- 기존 방식에서의 변화
    - 기존에는 `SetGraphicsRootConstantBufferView()`를 사용해 Root Signature에서 직접 CBV를 참조했다.
    - 이 방식은 `SetGraphicsRootDescriptorTable()`을 사용해 Descriptor Heap의 특정 CBV를 선택한다.
    - 즉, CBV를 직접 등록하는 게 아니라 Descriptor Heap을 통해 간접적으로 참조한다. 

&nbsp;

- `CommandList::SetgraphicsRootDescriptorTable()`
    - 참고: [MS_CommandList::SetDescriptorHeaps](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)
    - 역할
        - Descriptor Table에서 배열 중 몇 번째 것을 사용할지 설정하는 함수 
    - 특징
        - Root Signature에서 설정한 Descriptor Table을 사용할 때 호출한다. 
        - Descriptor Heap 내부의 특정 CBV/SRV/UAV를 선택하여 쉐이더에서 사용할 수 있도록 한다. 
- `CommandList::SetDescriptorHeaps()`
    - 참고: [MS_CommandList::SetDescriptorHeaps()](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-setdescriptorheaps)
    - 역할
        - 어떤 Descriptor Heap을 사용할 건지 매핑하는 함수
    - 특징
        - Descriptor Heap을 쉐이더에서 사용할 수 있도록 설정한다.
        - CBV/SRV/UAV Heap과 Sampler Heap을 한 번에 하나씩만 바인딩 할 수 있다.
        - 한 번 설정하면 여러 번 사용할 수 있기 때문에 너무 자주 호출하지 않도록 주의해야 한다. (호출 비용이 크므로 남발하지 말 것)
        - CPU에서 사용하는 Descriptor Heap과 다르게, 쉐이더에서 사용할 Heap은 반드시 Shader Visible 옵션을 활성화 해야 한다.
        - 참고. Descriptor Heap을 여러 개 만드는 게 아니라, 한 개를 크게 만들어서 사용한다.

---

## Root Sinature 수정

Root Signature를 설정할 때 Descriptor Table을 추가해보자.

&nbsp;

- Root Signature 설정 (`RootSignature::Init()`)
    1. CBV_REGISTER enum을 정의해 레지스터를 명확하게 관리한다.
        ```cpp
        enum class CBV_REGISTER
        {
            b0,
            b1,
            b2,
            b3,
            b4,
        
            END // 참고. END는 enum의 총 개수
        };
        
        enum
        {
            SWAP_CHAIN_BUFFER_COUNT = 2,
            CBV_REGISTER_COUNT = CBV_REGISTER::END,
            REGISTER_COUNT = CBV_REGISTER::END,
        };
        ```
    2. `CD3DX12_DESCRIPTOR_RANGE`를 사용해 Descriptor Table을 설정한다.
    3. Root Parameter를 Descriptor Table로 설정한다.
    4. `D3D12SerializeRootSignature()`를 사용해 Root Signature를 생성한다.

```cpp
CD3DX12_DESCRIPTOR_RANGE ranges[] =
{
    CD3DX12_DESCRIPTOR_RANGE(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, CBV_REGISTER_COUNT, 0), // b0~b4
};

CD3DX12_ROOT_PARAMETER param[1];
param[0].InitAsDescriptorTable(_countof(ranges), ranges);

D3D12_ROOT_SIGNATURE_DESC sigDesc = CD3DX12_ROOT_SIGNATURE_DESC(_countof(param), param);
sigDesc.Flags = D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT; // 입력 조립기 단계

ComPtr<ID3DBlob> blobSignature;
ComPtr<ID3DBlob> blobError;
::D3D12SerializeRootSignature(&sigDesc, D3D_ROOT_SIGNATURE_VERSION_1, &blobSignature, &blobError);
device->CreateRootSignature(0, blobSignature->GetBufferPointer(), blobSignature->GetBufferSize(), IID_PPV_ARGS(&_signature));
```

- 최종적으로
    1. b0 ~ b4의 CBV를 Descriptor Table을 통해 참조할 수 있도록 설정했다.
    2. CBV를 Root Signature에 직접 넣지 않고, Descriptor Table을 통해 접근하게 되었다.

---

## Descriptor Heap 생성

Root Signature를 설정했으므로, 이제 Constant Buffer에서 Descriptor Heap을 생성하여 CBV를 저장해야 한다.

&nbsp;

- Descriptor Heap 생성 (`ConstantBuffer::CreateView()`)
    1. 멤버 변수 추가
        ```cpp
        ComPtr<ID3D12DescriptorHeap>    _cbvHeap;   
	    D3D12_CPU_DESCRIPTOR_HANDLE	    _cpuHandleBegin = {};   // 시작 핸들 주소
	    uint32  _handleIncrementSize = 0; 
        
        // e.x. i번 핸들을 구하고 싶으면, _cpuHandleBegin += i * handleIncrementSize
        ```
    2. `D3D12_DESCRIPTOR_HEAP_DESC`를 사용하여 CBV를 저장할 Descriptor Heap을 생성한다.
    3. CPU Handle을 저장하여 특정 인덱스의 CBV를 참조할 수 있도록 설정한다.
    4. 각 CBV에 대해 `CreateConstantBufferView()`를 호출하여 Descriptor Heap에 등록한다.

```cpp
D3D12_DESCRIPTOR_HEAP_DESC cbvDesc = {};
cbvDesc.NumDescriptors = _elementCount;
cbvDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
cbvDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
DEVICE->CreateDescriptorHeap(&cbvDesc, IID_PPV_ARGS(&_cbvHeap));

_cpuHandleBegin = _cbvHeap->GetCPUDescriptorHandleForHeapStart();
_handleIncrementSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);

for (uint32 i = 0; i < _elementCount; ++i)
{
    D3D12_CPU_DESCRIPTOR_HANDLE cbvHandle = GetCpuHandle(i);

    D3D12_CONSTANT_BUFFER_VIEW_DESC cbvDesc = {};
    cbvDesc.BufferLocation = _cbvBuffer->GetGPUVirtualAddress() + static_cast<uint64>(_elementSize) * i;
    cbvDesc.SizeInBytes = _elementSize;

    DEVICE->CreateConstantBufferView(&cbvDesc, cbvHandle);
}
```

- 최종적으로 
    1. CBV Descriptor Heap이 생성되고, 각 CBV가 Constant Buffer를 가리키게 된다.
    2. 이제 Shader에서 Descriptor Heap을 통해 CBV를 사용할 수 있다.

---

## Descriptor Table과 Root Signature 연결

이제 Descriptor Table과 Root Signature를 연결해보자.

1. `ConstantBuffer::Init()`에서 `ConstantBuffer::CreateView()`를 호출하여 CBV Descriptor Heap을 설정한다.
    ```cpp
    void ConstantBuffer::Init(uint32 size, uint32 count)
    {
        _elementSize = (size + 255) & ~255;
        _elementCount = count;

        CreateBuffer();
        CreateView(); // CBV Descriptor Heap 생성
    }
    ```
2. `ConstantBuffer::GetCpuHandle()` 함수를 추가해 특정 인덱스 핸들을 가져오도록 한다.
    ```cpp
    D3D12_CPU_DESCRIPTOR_HANDLE ConstantBuffer::GetCpuHandle(uint32 index)
    {
        return CD3DX12_CPU_DESCRIPTOR_HANDLE(_cpuHandleBegin, index * _handleIncrementSize);
    }
    ```

- 최종적으로 CBV Descriptor Heap이 각 Constant Buffer를 참조하고, Root Signature에서 Descriptor Table을 통해 CBV Descriptor Heap을 Shader에 전달하는 구조가 완성된다. 

![Image](https://github.com/user-attachments/assets/a5c4782c-d7df-4e08-be4f-6ba52376d441)

---

## Table Descriptor Heap 생성

여러 개의 Descriptor를 관리하는 Table Descriptor Heap class를 구현해보자. 여러 개의 그룹을 만들 수 있도록 `groupSize`와 `groupCount`를 사용한다.

&nbsp;

- 사용 변수 

```cpp
private:
	ComPtr<ID3D12DescriptorHeap> _descHeap;
	uint64					_handleSize = 0;
	uint64					_groupSize = 0;
	uint64					_groupCount = 0;
	uint32					_currentGroupIndex = 0;  // 현재 그룹 인덱스
```

&nbsp;

- Heap 초기화 (`TableDescriptorHeap::Init()`)
    1. (그룹 수 * 각 그룹의 레지스터 수)만큼 생성한다.
    2. `D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE` 설정한다. (GPU에서 접근 가능)
    3. `CreateDescriptorHeap()`을 호출해 Heap을 생성한다.
    4. 핸들 크기 및 그룹 크기를 저장한다.

```cpp
_groupCount = count;

D3D12_DESCRIPTOR_HEAP_DESC desc = {};
desc.NumDescriptors = count * REGISTER_COUNT;
desc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
desc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;

DEVICE->CreateDescriptorHeap(&desc, IID_PPV_ARGS(&_descHeap));

_handleSize = DEVICE->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
_groupSize = _handleSize * REGISTER_COUNT;
```

&nbsp;

- 인덱스 초기화 (`TableDescriptorHeap::Clear()`)
    - 매 프레임마다 그릅 인덱스를 초기화할 수 있도록 Clear 함수를 구현한다.

```cpp
	_currentGroupIndex = 0;
```

&nbsp;

- CBV 복사 (`TableDescriptorHeap::SetCBV()`)
    - CBV를 Descriptor Heap으로 복사한다.

```cpp
void TableDescriptorHeap::SetCBV(D3D12_CPU_DESCRIPTOR_HANDLE srcHandle, CBV_REGISTER reg)
{
	D3D12_CPU_DESCRIPTOR_HANDLE destHandle = GetCPUHandle(reg);

	uint32 destRange = 1;
	uint32 srcRange = 1;
	DEVICE->CopyDescriptors(1, &destHandle, &destRange, 1, &srcHandle, &srcRange, D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV);
}
```

&nbsp;

- CPU 핸들 반환 (`TableDescriptorHeap::GetCPUHandle()`)
    - Descriptor Heap에서 원하는 위치의 핸들을 가져온다.
        1. 현재 그룹 위치로 이동한다.
        2. 레지스터 위치로 이동한다.

```cpp
D3D12_CPU_DESCRIPTOR_HANDLE TableDescriptorHeap::GetCPUHandle(uint32 reg)
{
	D3D12_CPU_DESCRIPTOR_HANDLE handle = _descHeap->GetCPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	handle.ptr += reg * _handleSize;
	return handle;
}
```

&nbsp;

- Descriptor Heap을 레지스터에 전달 (`TableDescriptorHeap::CommitTable()`)
    - Descriptor Table을 Root Signature에 바인딩한다.
    - `_currentGroupIndex`를 이동해 다음 그룹을 사용할 준비를 한다.

```cpp
	D3D12_GPU_DESCRIPTOR_HANDLE handle = _descHeap->GetGPUDescriptorHandleForHeapStart();
	handle.ptr += _currentGroupIndex * _groupSize;
	CMD_LIST->SetGraphicsRootDescriptorTable(0, handle);

	_currentGroupIndex++;
```

![Image](https://github.com/user-attachments/assets/064f04dd-2384-435b-9256-f8d5159df939)

---

## 사용하기

Enginer과 Command Queue에서 Table Descriptor Heap을 사용할 수 있도록 Engine 클래스를 수정한다.

&nbsp;

- Engine
    - 멤버 변수 추가    
        ```cpp
        shared_ptr<TableDescriptorHeap> _tableDescHeap;
        ```
    - 멤버 함수 추가
        ```cpp
        shared_ptr<TableDescriptorHeap> GetTableDescHeap() { return _tableDescHeap; }
        ```
    - 초기화 함수 추가 (`Engine::Init()`)
        ```cpp
        _tableDescHeap = make_shared<TableDescriptorHeap>();
        _tableDescHeap->Init(256);
        ```
- CommandQueue
    - `RenderBegin()`에서 `Clear()` 및 `SetDescriptorHeaps()` 호출하기
        ```cpp
        GEngine->GetCB()->Clear();
        GEngine->GetTableDescHeap()->Clear();

        ID3D12DescriptorHeap* descHeap = GEngine->GetTableDescHeap()->GetDescriptorHeap().Get();
        _cmdList->SetDescriptorHeaps(1, &descHeap);
        ```

&nbsp;

Constant Buffer View 관련 코드를 수정해보자.

- `ConstantBufferView::PushData()` 수정
    - Root Descriptor 관련 코드를 삭제한다.
    - CBV를 반환하도록 변경한다. (`void` → `D3D12_CPU_DESCRIPTOR_HANDLE`)

```cpp
assert(_currentIndex < _elementSize);

::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

D3D12_CPU_DESCRIPTOR_HANDLE cpuHandle = GetCpuHandle(_currentIndex);

_currentIndex++;

return cpuHandle;
```

&nbsp;

- CBV 설정 후 `CommitTable()`를 호출 (`Mesh::Render()`) 
    1. Buffer에 데이터를 세팅한다.
    2. Buffer의 주소를 register에 전송한다.
    3. 모두 세팅이 끝났으면 TableDescHeap을 커밋한다.

```cpp
CMD_LIST->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
CMD_LIST->IASetVertexBuffers(0, 1, &_vertexBufferView);

{
	D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
	GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b0);
}
{
	D3D12_CPU_DESCRIPTOR_HANDLE handle = GEngine->GetCB()->PushData(0, &_transform, sizeof(_transform));
	GEngine->GetTableDescHeap()->SetCBV(handle, CBV_REGISTER::b1);
}

GEngine->GetTableDescHeap()->CommitTable();

CMD_LIST->DrawInstanced(_vertexCount, 1, 0, 0);
```

---

## 삼각형 그리기 

위에 작성한 코드대로 실행하면 삼각형이 잘 그려진다! 그려지는 화면은 저번 수업과 같지만, 저번에는 Root Descriptor 방식으로 삼각형을 그렸고, 이번에는 Descriptor Table 방식으로 삼각형을 그린 것이다.

![Image](https://github.com/user-attachments/assets/ce40bba7-346f-4dc8-9f95-d9c0401d3d5a)

---

[Root Signature 커밋](https://github.com/chaeeun-dev/DirectX12/commit/f46e2445e082feed3c0ca28bb47ba90f822ed2fd)

---

## 마치며

이번 시간에 배운 것
- Root Signature를 Root Table 방식으로 변경하기
- Table Descriptor Heap을 사용해 CBV를 관리하기
- CBV를 Descriptor Heap에 복사 후, Root Descriptor Table로 바인딩하기
- Mesh에서 CBV를 사용해 데이터를 렌더링하기
- 💡 즉시 실행되는 부분과 나중에 실행되는 부분을 이해하는 것이 중요함!

&nbsp;

어지럽다.. 뒤돌면 까먹으니,, 종이에 그림을 그려가며 복습해야겠다. 완벽하게 이해하자!! 

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*