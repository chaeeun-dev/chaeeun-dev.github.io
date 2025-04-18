---
title: "[DirectX 12] #1-3. 렌더링 파이프라인 OT"
excerpt: "GPU에 대한 이해와 렌더링 파이프라인"

categories:
  - DirectX12
tags:
  - [DirectX, RenderingPipeline]

permalink: /DirectX12/rendering-pipeline/

toc: true
toc_sticky: true

date: 2025-02-19
last_modified_at: 2025-02-19
---

## 들어가며

게임 세계에서는 수많은 연산이 필요하다. 특히 물체의 위치와 색상 변경, 게임 규칙 적용, 캐릭터 이동과 2D 화면 출력 등 다양한 작업이 수행된다.

이번 수업에서는 게임에서의 연산을 어떻게 처리하는지 간략하게 알아본다.

---

### CPU vs GPU

- CPU(중앙처리 장치)
	- 복잡한 연산을 수행하는 핵심 역할을 담당한다(고급&소수 인력).
	- 연산량이 많지 않은 경우에 그래픽 연산도 가능하다.
- GPU(그래픽처리 장치)
	- 그래픽과 관련된 연산을 전담하여 CPU의 부담을 줄인다.
	- 정점(삼각형 단위의 기본 요소) 연산을 병렬로 처리한다.
	- 연관성이 없는 독립적인 연산에 특화되어 있다.
![CpuGpuStructure](/assets/images/post_img/directx/CpuGpuStructure.png)



| 구분     | CPU                  | GPU                 |
| ------ | -------------------- | ------------------- |
| **특징** | 복잡한 연산 수행, 기억력 강함    | 단순 연산을 병렬로 빠르게 처리   |
| **구조** | 적은 수의 강력한 ALU(연산 유닛) | 대부분이 ALU로 구성됨       |
| **용도** | 게임 로직, AI, 일반 연산     | 그래픽 연산, AI, 비트코인 채굴 |
| **장점** | 범용적이고 정교한 계산 가능      | 병렬 연산에 특화됨          |


- 왜 게임의 연산을 GPU에 맡길까?
	- GPU는 정점(삼각형) 단위의 연산을 병렬 처리하여 대량의 그래픽 연산을 빠르게 수행한다.
	- CPU가 직접 모든 정점 연산을 하면 성능이 저하되어, GPU에 분담하여 최적화 하는 것.

---

## 렌더링 파이프라인

GPU가 그래픽 연산을 처리하는 단계적 과정으로, 개발자들이 연구한 결과 정형화된 프로세스다.
1. Input Assembler Stage 
	- CPU가 GPU에게 정점 데이터를 넘겨주는 단계
2. Vertex Shader Stage
	- 개별 정점(삼각형의 꼭짓점)의 좌표 변환을 수행하는 단계
3. Tessellation 과정
	- 정점을 추가하여 더 정밀한 그래픽을 구현함
	- Hull Shader → Tessellator → Domain Shader 단계를 거침
	- 지형이나 정밀한 오브젝트 표현에 유용함
4. Geometry Shader Stage
	- 정점을 추가
	- 작은 단위 오브젝트 표현에 유용
5. Rasterizer Stage(중요!)
	- 정점으로 이루어진 삼각형을 픽셀 단위로 변환하고 보간하는 단계
6. Pixel Shader Stage
	- 각 픽셀에 색을 입히거나 변경하는 단계
7. Output-Merger Stage
	- 최종적으로 렌더링 될 픽셀 색을 생성하는 단계

---

## DirectX의 역할

각 GPU 마다 사양이 다 다른데, GPU에 값을 어떻게 넘길까?
- DirectX는 GPU를 위한 표준 API 역할을 수행한다.
- 개발자는 DirectX를 통해 GPU에 작업을 요청하고, 제조사들은 DirectX 스펙이 맞게 GPU를 제작한다.
- 즉, DirectX를 사용하면 특정 제조사에 종속되지 않고, 다양한 GPU에서 동일한 방식으로 그래픽을 처리할 수 있다.

---

## 마치며

게임에서 GPU가 맡는 역할과 렌더링 파이프라인의 개념을 배웠다. 그리고 DirectX가 어떤 역할을 하는지 전체적인 숲을 살펴보았다.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*