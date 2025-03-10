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

## 지난 수업 복습

지난 수업에서 각 장치를 클래스로 나누어 관리했는데, 일부 책이나 강의에서는 이를 한 번에 관리하는 방식도 사용한다. 클래스로 따로 관리하면 컴포넌트 구조 및 종속 관계(예. Swap Cahin과 Descriptor Heap)를 이해하는 데 도움이 된다.

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
ComPtr<ID3D12CommandList> GetCmdList() { return _cmdList; }
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

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*