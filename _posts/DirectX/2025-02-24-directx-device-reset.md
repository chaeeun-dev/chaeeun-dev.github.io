---
title: "[DirectX 12] #2-2. 장치 초기화"
excerpt: "Device, CommandQueue, SwapChain, DescriptorHeap"

categories:
  - DirectX12
tags:
  - [DirectX, Device, DeviceReset]

permalink: /DirectX12/device-reset/

toc: true
toc_sticky: true

date: 2025-02-24
last_modified_at: 2025-02-27
---

## 들어가며

장치 초기화란 GPU와 DirectX를 연결하고, 그래픽 명령을 처리할 수 있도록 준비하는 과정이다. 

이번에는  `Device`, `Command Queue`, `Swap Chain`, `Descriptor Heap` 등 장치 초기화에 필요한 API의 개념을 이해하고, C++ 코드로 구현해볼 것이다.

> 💡 API란?
>
> Application Programming Interface의 줄임말이다. API 맥락에서 애플리케이션이라는 단어는 고유한 기능을 가진 모든 소프트웨어를 나타낸다. 인터페이스는 두 애플리케이션 간의 서비스 계약이라고 할 수 있다. 
> 이 계약은 요청과 응답을 사용하여 두 애플리케이션이 서로 통신하는 방법을 정의한다. 
>
> _출처:_ _[aws](https://aws.amazon.com/ko/what-is/api/#:~:text=API%EB%8A%94%20Application%20Programming%20Interface,%EA%B3%84%EC%95%BD%EC%9D%B4%EB%9D%BC%EA%B3%A0%20%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.)_

---

## 장치 초기화

`Device`, `Command Queue`, `Swap Chain`, `Descriptor Heap`과 이들을 한 곳에서 관리하는 `Engine`을 클래스로 구현한다.

---

### Device 

- Device는 인력 사무소 역할이라고 할 수 있는데, GPU와 연결하고 각종 객체를 생성하는 역할을 담당한다.
> Device가 GPU를 제어할 수 있는 이유 : COM(Component Object Model)
>    - COM은 DirectX에서 프로그래밍 언어 독립성과 하위 호환성을 제공하는 기술이다.
>    - DirectX는 COM 객체를 사용하여 GPU와 상호작용 한다.
>    - COM 객체의 내부 동작은 숨겨져 있기 때문에 개발자는 세부사항을 몰라도 된다.
>    - ComPtr(스마트 포인터)를 사용해 GPU 자원을 자동으로 관리한다(delete 신경쓰지 않아도 됨).
>    - → Device는 COM을 통해 GPU에 접근하고, 그래픽 객체를 생성할 수 있다.

- 사용 변수

```cpp
ComPtr<ID3D12Debug> _debugController; // 디버깅 활성화
ComPtr<IDXGIFactory> _dxgi; // 화면 관련 기능들
ComPtr<ID3D12Device> _device; // 각종 객체 생성
```

&nbsp;

- 초기화 (`Device::Init()`)
	1. 디버깅 활성화
		- 개발 및 디버깅을 위한 기능이다.
		- DirectX 12 API를 사용할 때 잘못된 사용법이나 오류를 출력창에 경고 메시지로 제공한다.
	2. DXGI 팩토리 생성 
		- DXGI(DirectX Graphis Infrastructure)은 Direct3D와 함께 쓰이는 API로, 디스플레이 관련 기능을 담당한다.
		- 전체 화면 모드 전환 및 창 모드 전환 지원
		- 현재 시스템에서 사용 가능한 그래픽 카드 및 디스플레이 모드 조회
	3. Direct3D 장치 생성

```cpp
::D3D12GetDebugInterface(IID_PPV_ARGS(&_debugController));
_debugController->EnableDebugLayer();

::CreateDXGIFactory(IID_PPV_ARGS(&_dxgi));

::D3D12CreateDevice(nullptr, D3D_FEATURE_LEVEL_11_0, IID_PPV_ARGS(&_device));
```

---

### Command Queue

- GPU에 작업을 요청하는 명령 큐(Command Queue)를 관리한다.
- 하나씩 요청하면 비효율적이기 때문에, 명령을 한 번에 모아 실행한다(외주 목록에 일감을 차곡차곡 기록했다가 한 번에 요청함). 
- `Fence`를 활용하여 CPU와 GPU를 동기화한다. 

&nbsp;

- 사용 변수

```cpp
ComPtr<ID3D12CommandQueue>	_cmdQueue;
ComPtr<ID3D12CommandAllocator>	_cmdAlloc;
ComPtr<ID3D12GraphicsCommandList>	_cmdList;

ComPtr<ID3D12Fence>	_fence;
uint32	_fenceValue = 0;
HANDLE	_fenceEvent = INVALID_HANDLE_VALUE;

shared_ptr<SwapChain>	_swapChain;
shared_ptr<DescriptorHeap>	_descHeap;
```

&nbsp;

- 초기화 (`CommandQueue::Init()`)
	1. GPU가 실행할 명령 리스트를 담을 `Command Queue`를 생성한다.
	2. 명령 리스트 할당을 위한 `Command Allocator`를 생성한다.
	3. 실제 GPU 작업을 담는 리스트인 `Command List`를 생성한다.
	4. CPU와 GPU 동기화 장치인 `Fence`를 생성한다.

```cpp
device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&_cmdQueue));
device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&_cmdAlloc));
device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, _cmdAlloc.Get(), nullptr, IID_PPV_ARGS(&_cmdList));
device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&_fence));
_fenceEvent = ::CreateEvent(nullptr, FALSE, FALSE, nullptr);
```

&nbsp;

- 동기화 (`CommandQueue::waitSync()`)
	- GPU 작업이 끝날 때까지 CPU가 대기하도록 설정한다.

```cpp
_fenceValue++;
_cmdQueue->Signal(_fence.Get(), _fenceValue);
if (_fence->GetCompletedValue() < _fenceValue)
{
	_fence->SetEventOnCompletion(_fenceValue, _fenceEvent);
	::WaitForSingleObject(_fenceEvent, INFINITE);
}
```

&nbsp;

- 렌더링 준비 (`CommandQueue::RenderBegin()`)
	- GPU가 렌더링할 준비를 하도록 설정하는 함수이다.
	- 후면 버퍼를 렌더 타겟으로 변경하고 렌더링을 준비한다.
	1. `Command Allocator`와 `Command List`를 리셋한다.
	2. 현재 화면에 출력(Present) 중이던 후면 버퍼를 RTV로 변경한다.
	3. 뷰포트(GPU가 그릴 화면 크기와 위치)와 시저(화면 잘라 특정 영역만 렌더링)를 설정한다.
	4. 후면 버퍼에 RTV를 지정하고, 후면 버퍼를 GPU의 렌더 타겟으로 설정한다.

```cpp
_cmdAlloc->Reset();
_cmdList->Reset(_cmdAlloc.Get(), nullptr);

D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(
_swapChain->GetCurrentBackBufferResource().Get(),
D3D12_RESOURCE_STATE_PRESENT, // 화면 출력
D3D12_RESOURCE_STATE_RENDER_TARGET); // 외주 결과물
_cmdList->ResourceBarrier(1, &barrier);

_cmdList->RSSetViewports(1, vp);
_cmdList->RSSetScissorRects(1, rect);

D3D12_CPU_DESCRIPTOR_HANDLE backBufferView = _descHeap->GetBackBufferView();
_cmdList->ClearRenderTargetView(backBufferView, Colors::LightSteelBlue, 0, nullptr);
_cmdList->OMSetRenderTargets(1, &backBufferView, FALSE, nullptr);
```

&nbsp;

- 렌더링 끝 화면 출력 (`CommandQueue::RenderEnd()`)
	- GPU가 렌더링을 마치고 화면에 출력하는 함수이다.
	- 후면 버퍼를 화면에 출력하고 GPU가 명령을 실행한다.
	1. 후면 버퍼를 다시 화면 출력(Present) 가능 상태로 변경한다.
	2. 커맨드 리스트를 닫는다. (닫아야 GPU 실행 가능)
	3. GPU가 실행할 수 있도록 커맨드 리스트를 실행 큐에 등록한다.
	4. 후면 버퍼를 전면 버퍼(현재 화면)으로 변경한다. (렌더링이 끝난 이미지를 화면에 표시함)
	5. CPU가 GPU의 작업이 끝날 때까지 대기한다.
	6. 다음 프레임에서 사용할 후면 버퍼로 변경하기 위해 스왑체인 인덱스를 변경한다.

```cpp
D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(
	_swapChain->GetCurrentBackBufferResource().Get(),
	D3D12_RESOURCE_STATE_RENDER_TARGET, // 외주 결과물
	D3D12_RESOURCE_STATE_PRESENT); // 화면 출력

_cmdList->ResourceBarrier(1, &barrier);
_cmdList->Close();

ID3D12CommandList* cmdListArr[] = { _cmdList.Get() };
_cmdQueue->ExecuteCommandLists(_countof(cmdListArr), cmdListArr);

_swapChain->Present();

WaitSync();

_swapChain->SwapIndex();

```

---

### Swap Chain

- Swap Chain은 GPU가 계산한 렌더링 결과물을 화면에 출력하는 과정을 관리한다.
- 더블 버퍼링을 통해 끊김 없는 화면 전환을 수행한다.
- DirectX에서는 `DXGI`가 스왑 체인을 관리한다.

>외주 작업과 결과물 전달 과정
>- 게임 화면을 GPU에 맡긴다(외주 요청).
>- GPU가 작업한 결과물을 특정 버퍼(`_renderTargets`)에 저장한다.
>- 버퍼에 저장된 내용을 화면에 출력한다.

- 더블 버퍼링(Dubble Buffering)
	- 전면 버퍼 : 현재 화면에 출력 중인 프레임
	- 후면 버퍼 : GPU가 작업 중인 프레임
	- 두 개의 버퍼를 번갈아 사용한다.

&nbsp;

- 사용 변수

```cpp
ComPtr<IDXGISwapChain> _swapChain;   // 스왑 체인 객체 
ComPtr<ID3D12Resource> _renderTargets[SWAP_CHAIN_BUFFER_COUNT];
uint32 _backBufferIndex = 0;   // 현재 GPU가 작업 중인 백버퍼 인덱스
```

&nbsp;

- 초기화 (`SwapChain::Init()`)
	1. 스왑체인 객체를 초기화한다.
	2. `DXGI_SWAP_CHAIN_DESC` 구조체를 설정하여 해상도, 버퍼 개수, 갱신 빈도 등을 정의한다.
	3. 스왑 체인 객체를 생성한다.
	4. 생성된 스왑 체인에 `Render Target 버퍼`를 할당한다.

```cpp
_swapChain.Reset();

DXGI_SWAP_CHAIN_DESC sd;
// sd 구조체 변수 설정하는 코드(생략)

dxgi->CreateSwapChain(cmdQueue.Get(), &sd, &_swapChain);

for (int32 i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
	_swapChain->GetBuffer(i, IID_PPV_ARGS(&_renderTargets[i]));
```

&nbsp;

- 화면 출력 (`SwapChain::Present()`)
	- GPU가 처리한 후면 버퍼를 화면에 출력한다(전면 버퍼로 전환).

```cpp
_swapChain->Present(0, 0);
```

&nbsp;

- 버퍼 인덱스 변경 (`SwapChain::SwapIndex()`)
	- 현재 GPU가 작업 중인 버퍼(후면 버퍼)를 변경한다.
	- `SWAP_CHAIN_BUFFER_COUNT` 2개를 기준으로 0과 1을 반복한다.

```cpp
_backBufferIndex = (_backBufferIndex + 1) % SWAP_CHAIN_BUFFER_COUNT;
```

---

### Descriptor Heap

- 기안서를 작성하는 것과 같다고 할 수 있는데, GPU가 사용할 리소스를 알아볼 수 있도록 정리하는 과정이다.
- DX11에서 `View`라고 불리던 개념이 DX12에서 `Descriptor`로 통합되었다.
- 디스크립터 힙은 디스크립터(뷰)들의 배열이다.
- DX12에서는 모든 뷰가 디스크립터로 통합되어 디스크립터 힙에서 관리된다.

|DX11|DX12|
|---|---|
|RTV, DSV, CBV, SRV, UAV **(뷰 View)**|RTV, DSV, CBV, SRV, UAV **(디스크립터 Descriptor)**|
|개별적으로 관리됨|**디스크립터 힙을 사용하여 배열 형태로 관리**|


> 기안서를 작성하는 과정
> - 외주를 맡길 때 여러 정보를 아무렇게나 보내면 GPU가 리소스를 정확하게 처리할 수 없다.
> - 따라서, 각종 리소스를 어떤 용도로 사용할지 명확하게 정의하여 넘겨줘야 한다.

&nbsp;

- 사용 변수

```cpp
ComPtr<ID3D12DescriptorHeap>	_rtvHeap; // Render Target View를 저장하는 힙
uint32	_rtvHeapSize = 0; // RTV 힙의 크기
D3D12_CPU_DESCRIPTOR_HANDLE	_rtvHandle[SWAP_CHAIN_BUFFER_COUNT]; // RTV 핸들 배열

shared_ptr<class SwapChain>	_swapChain; // 스왑체인 객체
```

&nbsp;

- 참고
	- `CD3DX12_CPU_DESCRIPTOR_HANDLE`과 같이 `CD`가 붙은 변수는 **d3dx12.h**에 포함되어 있는 변수다.

&nbsp;

- 초기화 (`DescriptorHeap::Init()`)
	- RTV(Render Target View) 디스크립터 힙을 생성하고 설정하는 함수이다.
	1. 스왑체인 정보를 저장한다.
	2. RTV 디스크립터 크기를 가져온다.
	3. RTV 디스크립터 힙을 설정한다. (같은 종류의 데이터를 배열로 관리함)
	4. RTV 디스크립터 힙을 생성한다.
	5. 디스크립터 힙의 시작 주소를 가져온다.
	6. RTV 디스크립터를 할당 및 설정한다. (디스크립터 힙의 시작 주소 + (디스크립터 크기 * 인덱스))

```cpp
_swapChain = swapChain;

_rtvHeapSize = device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);

D3D12_DESCRIPTOR_HEAP_DESC rtvDesc;
rtvDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
rtvDesc.NumDescriptors = SWAP_CHAIN_BUFFER_COUNT;
rtvDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
rtvDesc.NodeMask = 0;

device->CreateDescriptorHeap(&rtvDesc, IID_PPV_ARGS(&_rtvHeap));	

D3D12_CPU_DESCRIPTOR_HANDLE rtvHeapBegin = _rtvHeap->GetCPUDescriptorHandleForHeapStart();

for (int i = 0; i < SWAP_CHAIN_BUFFER_COUNT; i++)
{
	_rtvHandle[i] = CD3DX12_CPU_DESCRIPTOR_HANDLE(rtvHeapBegin, i * _rtvHeapSize);
	device->CreateRenderTargetView(swapChain->GetRenderTarget(i).Get(), nullptr, _rtvHandle[i]);
}
```

&nbsp;

- 백 퍼버 RTV 핸들 가져오기 (`DescriptorHeap::GetBackBufferView`)
	- 현재 사용 중인 후면 버퍼의 RTV 핸들을 반환한다.

```cpp
return GetRTV(_swapChain->GetCurrentBackBufferIndex());
```

---

### Engine

- Engine 클래스는 DirectX 12의 핵심 API를 관리한다.

&nbsp;

- `EnginePch.h`에 윈도우 정보 구조체를 정의하여 창 크기 및 모드를 관리하고, `ResizeWindow()` 함수를 통해 초기화와 창 크기 변경할 수 있는 기능을 구현한다.

```cpp
struct WindowInfo
{
	HWND hwnd;	// 출력할 윈도우 핸들
	int32 width;	// 너비
	int32 height;	// 높이
	bool windowed;	// 창 모드 여부
};
```

&nbsp;

- `Engine` 클래스를 어디서든 접근할 수 있도록 하는 2가지 방법
1. 싱글톤 디자인 패턴 사용하기
2. 전역 클래스 사용하기 (`GEngine` 전역 객체 활용) → 프로젝트에서는 전역 클래스를 사용한다!

&nbsp;

- 초기화 (`Engine::Init()`)
	1. 윈도우 크기를 재설정한다.
	2. 그려질 화면의 크기를 설정한다.
	3. DX12의 핵심 객체들을 스마트 포인터로 생성한다.
	4. 생성된 객체들을 초기화한다.

```cpp
_window = info;
ResizeWindow(info.width, info.height);

_viewport = { 0, 0, static_cast<FLOAT>(info.height), 0.0f, 1.0f };
_scissorRect = CD3DX12_RECT(0, 0, info.width, info.height);

_device = make_shared<Device>();
_cmdQueue = make_shared<CommandQueue>();
_swapChain = make_shared<SwapChain>();
_descHeap = make_shared<DescriptorHeap>();

_device->Init();
_cmdQueue->Init(_device->GetDevice(), _swapChain, _descHeap);
_swapChain->Init(info, _device->GetDXGI(), _cmdQueue->GetCmdQueue());
_descHeap->Init(_device->GetDevice(), _swapChain);
```

&nbsp;

- 렌더링 (`Render()` - `RenderBegin()` & `RenderEnd()`)
	- 커맨드 큐를 이용하여 렌더링을 시작하고 끝낸다.

```cpp
void Engine::Render()
{
	RenderBegin();

	// TODO : 나머지 물체들 그려준다.

	RenderEnd();
}

void Engine::RenderBegin()
{
	_cmdQueue->RenderBegin(&_viewport, &_scissorRect);
}

void Engine::RenderEnd()
{
	_cmdQueue->RenderEnd();
}
```

---

## 화면 그리기

- 각 클래스들이 잘 만들었는지 확인하기 위해 빌드 후, Client 프로젝트의 `Game` 클래스에 Engine 프로젝트의 `GEngine`을 가져와 `Init()`과 `Update()`를 넣고 `Client.cpp`의 게임 루프에서 실행해본다.
- `GEngine`을 동적 할당으로 생성했는데도 empty가 떠서 헤맸는데, 빌드 순서를 `Engine 프로젝트`가 먼저 되도록 설정하니, 커맨드 큐에서 설정한 `Colors::LightSteelBlue` 하늘색 화면이 잘 출력됐다!

![DeviceResetResult](/assets/images/post_img/directx/DeviceResetResult.png)

---

## 마치며

너무 어렵고 머리가 아픈 수업이었다. 정리하는 데도 한 세월 걸렸다..ㅠ0ㅠ 그래도 나무까진 아니지만 숲은 보이는 기분이다. 이 포스트 계속 복습하고, 수업 듣고, 실습하다보면 언젠가는 쉬워지겠지!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*