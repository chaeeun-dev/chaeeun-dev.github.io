---
title: "[DirectX 12] #3-2. Material"
excerpt: "Material 클래스 구현하기"

categories:
  - DirectX12
tags:
  - [DirectX, Material]

permalink: /DirectX12/material/

toc: true
toc_sticky: true

date: 2025-04-22
last_modified_at: 2025-04-22
---

## 들어가며

앞선 수업에서는 Mesh, Shader, Texutre를 개별적으로 연결하여 렌더링했다. 하지만 게임의 규모가 커질수록 개별 관리가 비효율적이기 때문에, 공통 자원을 묶고 효율적으로 관리할 수 있는 구조가 필요하다. 여기서 등장하는 개념이 바로 Material이다.

---

## Material

- Material이란?
    - Mesh가 어떤 Shader를 사용할지, 어떤 Texutre를 바인딩할지, Shader에게 넘겨줄 값은 무엇인지 하나로 묶는 역할을 한다.
    - Mesh에 Material을 입히면, Shader와 Texture 등 각각의 인자들이 하나로 묶어진다.
    - Unity, Unreal Engine에서는 아래와 같이 툴로 활용되는데, Material을 설정하면 여러 값들을 하나로 묶을 수 있다.

![MaterialUnity](/assets/images/post_img/directx/MaterialUnity.png)

---

### Shader 파일

- `default.hlsl` 수정
    - b0: Transform 전용
    - b1: Material 전용

```hlsl
cbuffer TEST_B0 : register(b0)  // Transform용
{
    float4 offset0;
};

cbuffer MATERIAL_PARAMS : register(b1)  // Material용
{
    int int_0;
    int int_1;
    int int_2;
    int int_3;
    int int_4;
    float float_0;
    float float_1;
    float float_2;
    float float_3;
    float float_4;
};

Texture2D tex_0 : register(t0);
Texture2D tex_1 : register(t1);
Texture2D tex_2 : register(t2);
Texture2D tex_3 : register(t3);
Texture2D tex_4 : register(t4);
```

---

### ConstantBuffer 클래스

- ConstantBuffer 타입 추가
    ```cpp
    enum class CONSTANT_BUFFER_TYPE : uint8
    {
        TRANSFORM,
        MATERIAL,
        END
    };

    enum
    {
        CONSTANT_BUFFER_COUNT = static_cast<uint8>(CONSTANT_BUFFER_TYPE::END)
    }; 
    ``` 

&nbsp;

- `ConstantBuffer::Init()` 수정
    - 레지스터 번호 관련 멤버 변수 추가 `CBV_REGISTER _reg = {};`
    - 매개변수 인자로 받은 레지스터 번호로 _reg 초기화

&nbsp;

- `ConstantBuffer::PushData()` 수정
    - 반환형을 void로 수정
        ```cpp
        assert(_currentIndex < _elementCount);
        assert(_elementSize == ((size + 255) & ~255));

        ::memcpy(&_mappedBuffer[_currentIndex * _elementSize], buffer, size);

        D3D12_CPU_DESCRIPTOR_HANDLE cpuHandle = GetCpuHandle(_currentIndex);
        GEngine->GetTableDescHeap()->SetCBV(cpuHandle, _reg);

        _currentIndex++;
        ```

---

### Engine 클래스

- ConstantBuffer 관리 방식 변경
    - 기존 `shared_ptr<ConstantBuffer> _cb;` 코드 삭제
    - 새로운 `vector<shared_ptr<ConstantBuffer>> _constantBuffers;` 코드 추가
    - ConstantBuffer를 여러 개 사용하기 위해 변경한다.

&nbsp;

- `Engine::CreateConstantBuffer()` 함수 추가
    - 레지스터 번호를 받아 Constant Buffer를 생성한다.
    ```cpp
    void CreateConstantBuffer(CBV_REGISTER reg, uint32 bufferSize, uint32 count)
    {
        uint8 typeInt = static_cast<uint8>(reg);
        assert(_constantBuffers.size() == typeInt);

        shared_ptr<ConstantBuffer> buffer = make_shared<ConstantBuffer>();
        buffer->Init(reg, bufferSize, count);
        _constantBuffers.push_back(buffer);
    }
    ```

&nbsp;

- `Engine::Init()` 수정
    - Transform과 Material 관련 ConstantBuffer를 생성한다.
    ```cpp
    CreateConstantBuffer(CBV_REGISTER::b0, sizeof(Transform), 256);
    CreateConstantBuffer(CBV_REGISTER::b1, sizeof(MaterialParams), 256);       
    ```

---

### Material 클래스

Resource 폴더에 Material 클래스를 생성한다.

&nbsp;

- 상수 정의 및 구조체

```cpp
enum
{
	MATERIAL_INT_COUNT = 5,
	MATERIAL_FLOAT_COUNT = 5,
	MATERIAL_TEXTURE_COUNT = 5,
};

struct MaterialParams
{
	void SetInt(uint8 index, int32 value) { intParams[index] = value; }
	void SetFloat(uint8 index, float value) { floatParams[index] = value; }

	array<int32, MATERIAL_INT_COUNT> intParams;
	array<float, MATERIAL_FLOAT_COUNT> floatParams;
};
```

&nbsp;

- 멤버 변수

```cpp
shared_ptr<Shader> _shader;
MaterialParams _params;
array<shared_ptr<Texture>, MATERIAL_TEXTURE_COUNT> _textures;
```
    
&nbsp;

- 멤버 함수
    - `void SetShader(shared_ptr<Shader> shader) { _shader = shader; }`
    - `void SetInt(uint8 index, int32 value) { _params.SetInt(index, value); }`
    - `void SetFloat(uint8 index, float value) { _params.SetFloat(index, value); }`
    - `void SetTexture(uint8 index, shared_ptr<Texture> texture) { _textures[index] = texture; }`
    - `Update()`
        1. CBV 업로드
        2. SRV 업로드
        3. 파이프라인 설정
        ```cpp
        // CBV 업로드
        CONST_BUFFER(CONSTANT_BUFFER_TYPE::MATERIAL)->PushData(&_params, sizeof(_params));

        // SRV 업로드
        for (size_t i = 0; i < _textures.size(); i++)
        {
            if (_textures[i] == nullptr)
                continue;

            SRV_REGISTER reg = SRV_REGISTER(static_cast<int8>(SRV_REGISTER::t0) + i);
            GEngine->GetTableDescHeap()->SetSRV(_textures[i]->GetCpuHandle(), reg);
        }

        // 파이프라인 설정
        _shader->Update();
        ```

---

### CommandQueue 클래스

- `CommandQueue::RenderBegin()` 수정
    - Transform과 Material 관련 함수 추가
        ```cpp
        GEngine->GetConstantBuffer(CONSTANT_BUFFER_TYPE::TRANSFORM)->Clear();
        GEngine->GetConstantBuffer(CONSTANT_BUFFER_TYPE::MATERIAL)->Clear();
        GEngine->GetTableDescHeap()->Clear();
        ```

---

### Mesh 클래스

- Texture 관련 변수와 함수를 제거하고, Material 변수와 함수를 추가

```cpp
shared_ptr<Material> _mat;

void SetMaterial(shared_ptr<Material> mat) { _mat = mat; }
```

---

### Game 클래스

- `default.hlsl` 파일 수정
    - float 0, 1, 2를 사용하도록 쉐이더 파일을 수정한다.
```hlsl
VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = float4(input.pos, 1.f);

    output.pos.x += float_0;
    output.pos.y += float_1;
    output.pos.z += float_2;

    output.color = input.color;
    output.uv = input.uv;

    return output;
}
```

&nbsp;

- `Game::Init()` 수정
    1. Shader 생성 및 초기화
    2. Texture 생성 및 초기화
    3. Material 생성 및 설정
        - Shader 등록
        - Shader에 넘길 float 값 설정
            - x축으로 +0.3f
            - y축으로 +0.4f
            - z축으로 +0.3f
        - Texture 등록 (t0 레지스터에 연결되어 GPU에 업로드됨)
    4. Mesh에 Material 설정
        - 이후 `Mesh::Render()` 시, 이 Material이 자동으로 `Udpate()`되어, CBV, SRV, Shader가 한 번에 적용됨

```cpp
shared_ptr<Shader> shader = make_shared<Shader>();
shared_ptr<Texture> texture = make_shared<Texture>();
shader->Init(L"..\\Resources\\Shader\\default.hlsli");		
texture->Init(L"..\\Resources\\Texture\\dog.jpg");

shared_ptr<Material> material = make_shared<Material>();
material->SetShader(shader);
material->SetFloat(0, 0.3f);
material->SetFloat(1, 0.4f);
material->SetFloat(2, 0.3f);
material->SetTexture(0, texture);

mesh->SetMaterial(material);
```

&nbsp;

- 결과
    - 흰둥이가 그려진 Mesh가 (0.3, 0.4, 0.3) 방향으로 이동했다.

![MaterialResult](/assets/images/post_img/directx/MaterialResult.png)

---

[Material 커밋](https://github.com/chaeeun-dev/DirectX12/commit/5fa935a6dafba0b6c0a640bc026d31df17ad6e9f)

---

## 마치며

- 이번 시간에 배운 것
    - Material 개념과 클래스로 구현하기

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*