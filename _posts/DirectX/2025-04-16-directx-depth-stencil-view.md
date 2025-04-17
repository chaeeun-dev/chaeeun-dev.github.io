---
title: "[DirectX 12] #2-8. DepthStencilView"
excerpt: "카메라 좌표계를 투영 좌표게로 변환하기"

categories:
  - DirectX12
tags:
  - [DirectX, DepthStencilView]

permalink: /DirectX12/depth-stencil-view/

toc: true
toc_sticky: true

date: 2025-04-16
last_modified_at: 2025-04-16
---

## 들어가며

이번 시간에는 3D 객체를 화면에 출력하기 위해 꼭 필요한 Depth 개념을 알아보고, 픽셀의 깊이 값을 추적해 가장 가까운 객체의 픽셀만 출력되도록 처리하는 실습을 해본다.

---

## DepthStencil

- Depth
    - 3D 게임의 세상은 입체적이지만, 모니터에 출력되는 화면은 2D로 납작하다. 3D를 2D로 나타내기 위해 투영(Projection)이 필요하다.

> 💡 Projection이란?
>
> 카메라 좌표계를 투영 좌표계로 변환해 원근감을 적용한다.

&nbsp;

- Depth Buffer
    - 역할
        - 픽셀 단위로 Depth(깊이)값을 저장한다.
        - z값을 비교해 가장 가까운 픽셀만 화면에 출력한다.
    - 예시
        - 두 사각형이 겹칠 경우, 카메라에 가까운 사각형만 보인다.
    - 특징
        - 깊이 값은 일반적으로 0.0 ~ 1.0이다.
        - 1에 가까울 수록 멀리 있는 오브젝트이고, 1 이상은 아예 계산하지 않는다.

&nbsp;

- Stencil
    - 실습에서 사용하지 않지만, 개념적으로 알아보자.
    - 개념
        - 미술에서 판에 구멍을 뚫고 잉크를 통과시켜 찍어내는 공판화 기법이다.
        - 그래픽스에선 픽셀마다 스텐실 값을 기록하고, 특정 조건을 만족할 때만 해당 픽셀에 렌더링을 허용한다.
    - 활용 예시
        - 실루엣 처리, 미러 효과, 그림자 마스킹 등

---

### DepthStencilBuffer 클래스

- 멤버 변수
    - DepthStencilView 리소스와 뷰를 생성해 Depth 정보를 저장하는 버퍼를 추가한다.
```cpp
ComPtr<ID3D12Resource>				_dsvBuffer;
ComPtr<ID3D12DescriptorHeap>		_dsvHeap;
D3D12_CPU_DESCRIPTOR_HANDLE			_dsvHandle = {};
DXGI_FORMAT							_dsvFormat = {};
```

&nbsp;

- `DepthStencilView::Init()` 추가
    - window를 매개변수로 갖는 이유는 화면 크기(픽셀 수)에 따라 버퍼를 만들어줘야 하기 때문이다.
    - format `D3D12_HEAP_TYPE_DEFAULT`은 4byte를 모두 Depth 값으로 쓰겠다는 뜻이다. 
    1. 버퍼 크기 생성
    2. Heap 생성
    3. DepthStencilView 생성

```cpp
void DepthStencilBuffer::Init(const WindowInfo& window, DXGI_FORMAT dsvFormat)
{
	_dsvFormat = dsvFormat;

	D3D12_HEAP_PROPERTIES heapProperty = CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT);

	D3D12_RESOURCE_DESC desc = CD3DX12_RESOURCE_DESC::Tex2D(_dsvFormat, window.width, window.height);
	desc.Flags = D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL;

	D3D12_CLEAR_VALUE optimizedClearValue = CD3DX12_CLEAR_VALUE(_dsvFormat, 1.0f, 0);

	DEVICE->CreateCommittedResource(
		&heapProperty,
		D3D12_HEAP_FLAG_NONE,
		&desc,
		D3D12_RESOURCE_STATE_DEPTH_WRITE,
		&optimizedClearValue,
		IID_PPV_ARGS(&_dsvBuffer));

	D3D12_DESCRIPTOR_HEAP_DESC heapDesc = {};
	heapDesc.NumDescriptors = 1;
	heapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
	heapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_DSV;

	DEVICE->CreateDescriptorHeap(&heapDesc, IID_PPV_ARGS(&_dsvHeap));

	_dsvHandle = _dsvHeap->GetCPUDescriptorHandleForHeapStart();
	DEVICE->CreateDepthStencilView(_dsvBuffer.Get(), nullptr, _dsvHandle);
}
```

---

### Engine 클래스

이제 Engine 클래스에 DSV를 연동해보자.

&nbsp;

- 멤버 변수 및 함수 추가
```cpp
shared_ptr<DepthStencilBuffer> _depthStencilBuffer;
shared_ptr<DepthStencilBuffer> GetDepthStencilBuffer() { return _depthStencilBuffer; }
```

&nbsp;

- `Engine::Init()`에 DepthStencilBuffer 생성
```cpp
_depthStencilBuffer = make_shared<DepthStencilBuffer>();
ResizeWindow(info.width, info.height);
```

- `Engine::ResizeWindow()`에 DepthStencilBuffer 초기화
```cpp
_depthStencilBuffer->Init()
```

---

### Command Queue 클래스

- `CommandQueue::RenderBegin()`
    - DepthStencilView 설정 및 초기화
        - `OMSetRenderTargets()`로 RenderTarget과 함께 설정
        - `ClearDepthStencilView()`로 깊이를 초기화
        ```cpp
        D3D12_CPU_DESCRIPTOR_HANDLE depthStencilView = GEngine->GetDepthStencilBuffer()->GetDSVCpuHandle();
        _cmdList->OMSetRenderTargets(1, &backBufferView, FALSE, &depthStencilView);
        _cmdList->ClearDepthStencilView(depthStencilView, D3D12_CLEAR_FLAG_DEPTH, 1.0f, 0, 0, nullptr);
        ```

---

### Shader 클래스

- `default.hlsi` 파일 수정
    - `VS_MAIN()`에서 `output.pos`에 `offset0`을 더해 z값 변화를 반영함으로써 깊이 차이를 나타낸다. 
    ```hlsl
    VS_OUT VS_Main(VS_IN input)
    {
        VS_OUT output = (VS_OUT)0;

        output.pos = float4(input.pos, 1.f);
        output.pos += offset0;
        output.color = input.color;
        output.uv = input.uv;

        return output;
    }
    ```

&nbsp;

- `Shader::Init()`
    - pipeline에 DSV를 추가한다.
        1. `DepthStencilState` 기본값 사용
        2. `DSVFormat`을 파이프라인에 등록
    ```cpp
    _pipelineDesc.DepthStencilState = CD3DX12_DEPTH_STENCIL_DESC(D3D12_DEFAULT);
    _pipelineDesc.DSVFormat = GEngine->GetDepthStencilBuffer()->GetDSVFormat();
    ```

---

### Game 클래스

같은 위치의 두 Mesh를 z값만 다르게 설정하여 깊이 테스트를 해본다. z값이 더 작을 수록(앞에 있을수록) 화면에 그려진다.

&nbsp;

- case1
    - 오른쪽 Mesh의 z값에 0.2f 더한다.
    - 왼쪽 Mesh의 z값에 0.0f 더한다.
    - Depth 값이 더 작은(가까운) 왼쪽 Mesh가 앞에 그려진다.
    ![Result1](/assets/images/post_img/directx/DepthStencilBufferResult1.png)
- case2
    - 오른쪽 Mesh의 z값에 0.2f 더한다.
    - 왼쪽 Mesh의 z값에 0.3f 더한다.
    - Depth 값이 더 작은(가까운) 오른쪽 Mesh가 앞에 그려진다.
    ![Result2](/assets/images/post_img/directx/DepthStencilBufferResult2.png)

---

[DepthStencilView 커밋](https://github.com/chaeeun-dev/DirectX12/commit/566bb5302d7792f4a6e620be8095387e9e3f6674)

---

## 마치며

- 이번 시간에 배운 것
    - 3D 게임 세계를 2D 화면에 출력하기 위해 필요한 투영과 깊이에 대한 이해
    - Depth 값을 이용해 화면에 가까운 오브젝트를 그리는 방법 

&nbsp;

게임 수학에서 배운 깊이 버퍼를 코드로 구현해보니 신기했다. 사각형 두개 그리는데 이렇게 많은 코드를 수정해야하는데, 게임에선 얼마나 복잡할까..?????

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*