---
title: "[DirectX 12] #2-8. Root Signature"
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

## Rppt Sinature 설정

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*