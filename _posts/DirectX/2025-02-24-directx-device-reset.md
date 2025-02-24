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
last_modified_at: 2025-02-24
---

## 장치 초기화

장치 초기화란 GPU와 DirectX를 연결하고, 그래픽 명령을 처리할 수 있도록 준비하는 과정이다. 

이번에는 `Command Queue`, `Device`, `Swap Chain`, `Descriptor Heap` 등 장치 초기화에 필요한 API를 다뤄볼 것이다.

---

### Engine

- Engine 클래스는 DirectX 12의 핵심 API를 관리한다.
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
- `Engine` 클래스를 어디서든 접근할 수 있도록 설정한다.
1. 싱글톤 디자인 패턴 사용하기
2. 전역 클래스 사용하기 (`CEngine` 전역 객체 활용)

---

### Device 

- Device는 인력 사무소 역할이라고 할 수 있는데, GPU와 연결하고 각종 객체를 생성하는 역할을 담당한다.
- Device가 GPU를 제어할 수 있는 이유 : COM(Component Object Model)
    - COM은 DirectX에서 프로그래밍 언어 독립성과 하위 호환성을 제공하는 기술이다.
    - DirectX는 COM 객체를 사용하여 GPU와 상호작용 한다.
    - COM 객체의 내부 동작은 숨겨져 있기 때문에 개발자는 세부사항을 몰라도 된다.
    - ComPtr(스마트 포인터)를 사용해 GPU 자원을 자동으로 관리한다(delete 신경쓰지 않아도 됨).

→ Device는 COM을 통해 GPU에 접근하고, 그래픽 객체를 생성할 수 있다.

```cpp
	ComPtr<ID3D12Debug>			_debugController; // 디버기 활성화
	ComPtr<IDXGIFactory>		_dxgi; // 화면 관련 기능들
	ComPtr<ID3D12Device>		_device; // 각종 객체 생성
```

초기화 과정 (`Device::Init()`)
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

```cpp
device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&_cmdQueue));
device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&_cmdAlloc));
device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, _cmdAlloc.Get(), nullptr, IID_PPV_ARGS(&_cmdList));
device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&_fence));
_fenceEvent = ::CreateEvent(nullptr, FALSE, FALSE, nullptr);
```

동기화(`waitSunc()`)
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

---

### RenderBegin & RenderEnd

화면 렌더링에 관련된 기능인데, 이번 강의에선 다루지 않는다.

---


---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*