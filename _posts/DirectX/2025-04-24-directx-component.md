---
title: "[DirectX 12] #3-3. Component"
excerpt: "Component 개념 이해하고 객체에 부품 붙이기"

categories:
  - DirectX12
tags:
  - [DirectX, Component]

permalink: /DirectX12/component/

toc: true
toc_sticky: true

date: 2025-04-24
last_modified_at: 2025-04-24
---

## 들어가며

이번 시간에는 Component의 개념에 대해 알아보고 직접 클래스로 구현해본다.

---

## Component

- Unity vs Unreal 
    - Unity Engine
        - Component 기반으로, 객체는 여러 개의 Compoent로 구성된다. (Transform, MeshRenderer, Camera 등)
        - 부품처럼 조립하기 때문에 유지보수가 용이하고, 유연성이 높다.
    - Unreal Engine
        - 상속 기반으로, Actor → Pawn → Character 등으로 계층적 설계로 이루어진다.
        - 상속 트리를 기준으로 기능을 구현하기 때문에 구조적인 틀이 강하지만 유연성은 떨어진다.
- 이 수업은 Unity처럼 Component 기반으로 구현할 것이다.

---

### Component 클래스

그럼 Component를 코드로 직접 구현해보자.

Component 클래스는 모든 Component의 부모 클래스이고, GameObject와 연결되어 타입과 생명 주기 함수를 가진다.

&nbsp;

- Component Type

```cpp
enum class COMPONENT_TYPE : uint8
{
	TRANSFORM,
	MESH_RENDERER,
	// ...
	MONO_BEHAVIOUR,
	END,
};
```

&nbsp;

- 멤버 변수
    - `_type`: Component가 어떤 타입인지 구분한다.
    - `_gameObject`: 자신이 소유한 GameObject를 기리킨다
        - 💡 왜 `weak_ptr인가`? 
            1. 만약 `shared_ptr로` 연결하면 GameObject Component 간 순환 참조(circular reference)가 발생한다.
            2. 순환 참조는 reference count를 0으로 만들 수 없어 메모리 누수가 발생할 수 있다.
            3. 이를 방지하기 위해 Component에서는 GameObject를 `weak_ptr로` 참조한다.

```cpp
COMPONENT_TYPE _type;
weak_ptr<GameObject> _gameObject;
```

&nbsp;

- 생명 주기 함수 4종
    - `Awake`: 객체가 생성되고 나서 가장 먼저 호출되는데, 초기화 용도로 사용한다.
    - `Start`: Awake가 모두 끝나고 한 번 호출된다. 연관된 다른 Component와 상호작용할 준비가 됐을 때 사용한다.
    - `Update`: 매 프레임마다 호출된다.
    - `LateUpate`: Update가 모두 끝난 뒤 호출된다. Update와의 시점을 분리하고 싶을 때 사용한다.

```cpp
virtual void Awake() {}
virtual void Start() {}
virtual void Update() {}
virtual void LateUpdate() {}
```

&nbsp;

- Get, Set 함수
    - `IsValid()`: 이 Component가 유효한 GameObject를 가리키고 있는지 확인한다.
```cpp
COMPONENT_TYPE GetType() { return _type; }
bool IsValid() { return _gameObject.expired() == false; }

shared_ptr<GameObject> GetGameObject();
shared_ptr<Transform> GetTransform();
```

&nbsp;

- 내부 전용 함수 (GameObject만 접근 가능)
    - GameObject가 Component를 등록할 때만 `SetGameObject()`를 호출할 수 있도록 `friend`로 제한한다.
    - `shared_from_this()`를 통해 GameObject의 포인터를 전달받아 저장한다.

```cpp
friend class GameObject;

void SetGameObject(shared_ptr<GameObject> gameObject)
{
	_gameObject = gameObject;
}
```

---

### GameObject 클래스

Component들을 묶어 관리하는 최상위 객체로, 하나의 GameObject에는 여러 Component를 가질 수 있다.

이 클래스는 `enable_shared_from_this<GameObject>`를 상속받아 자기 자신을 `shared_ptr`로 전달할 수 있다. 이게 중요한 이유는 아래 `AddComponent()` 참고하기!

&nbsp;

- 멤버 변수
    - `_components`: 고정된 타입들을 배열(array)로 저장한다. Transform, MeshRenderer 등 기본 컴포넌트에 사용한다.
    - `_scripts`: 사용자가 직접 만든 스크립트를 저장하는 vector이다.

```cpp
array<shared_ptr<Component>, FIXED_COMPONENT_COUNT> _components;
vector<shared_ptr<MonoBehaviour>> _scripts;
```

&nbsp;

- `GameObject::Init()`
    - GameObject가 생성되면 Transform Component를 자동으로 추가한다. 모든 GameObject는 Transform을 기본으로 가지므로 초기화 과정에서 자동 생성하도록 한다.

```cpp
AddComponent(make_shared<Transform>());
```

&nbsp;

- `GameObject::AddComponent()`
    - `SetGameObject(shared_from_this())`
        - 자신의 스마트 포인터를 넘기기 위해 `shraed_from_this()`를 사용한다.
        - 만약 `make_shared<GameObject>(this)`처럼 사용하면 새로운 객체가 생성되므로 반드시 주의해야 한다.
    - `index < FIXED_COMPONENT_COUNT`
        - 정해진 기본 컴포넌트(Transform, MeshRenderer 등)인지 확인해 배열에 저장하고, 아니면 스크립트로 저장한다.
    - `dynamic_pointer_cast<MonoBehaviour>`
        - MonoBehaviour를 상속받는 사용자 정의 스크립트를 정확하게 캐스팅한다.

```cpp
component->SetGameObject(shared_from_this());

uint8 index = static_cast<uint8>(component->GetType());
if (index < FIXED_COMPONENT_COUNT)
{
    _components[index] = component;
}
else
{
    _scripts.push_back(dynamic_pointer_cast<MonoBehaviour>(component));
}
```

&nbsp;

- 생명 주기 함수 호출
    - Awake, Start, Update, Late 거의 동일한 구조이다.
    - 각각의 생명 주기 함수에 맞춰 연결된 모든 Component와 Script 함수가 실행된다.

```cpp
// Awake
for (shared_ptr<Component>& component : _components)
{
	if (component)
		component->Awake();
}

for (shared_ptr<MonoBehaviour>& script : _scripts)
{
	script->Awake();
}
```

&nbsp;

- GameObject::GetTransform()
    - Transform 타입의 Component가 어디 저장되어 있는지 인덱스를 통해 바로 찾아 반환한다.

```cpp
uint8 index = static_cast<uint8>(COMPONENT_TYPE::TRANSFORM);
return static_pointer_cast<Transform>(_components[index]);
```

---

### Transform & MonoBehaviour 클래스

클래스 틀만 작성하고 구현은 이번 시간에 하지 않는다. 

---

### MeshRenderer 클래스

- 멤버 변수 
    - Mesh와 Material 객체들을 스마트 포인터로 관리한다.
    - 각각 렌더링할 형상 데이터와 재질(쉐어더, 텍스처 등)을 담당한다.

```cpp
shared_ptr<Mesh> _mesh;
shared_ptr<Material> _material;
```

&nbsp;

- SetMesh, SetMaterial

```cpp
void SetMesh(shared_ptr<Mesh> mesh) { _mesh = mesh; }
void SetMaterial(shared_ptr<Material> material) { _material = material; }
```

&nbsp;

- `MeshRenderer::Update()`
    - 보통은 모든 컴포턴트의 Update 이후 Render를 따로 처리하는 게 일반적이다.
    - 하지만, 이 코드는 테스트 목적상 Update 안에 Render를 호출하고 있다.
    
```cpp
virtual void Update() override { Render(); }
```

&nbsp;

- `MeshRenderer::Render()`
    - 재질(Material)의 상태를 GPU에 반영한 후, 메쉬를 실제로 렌더링한다.
    - 원래는 `GetTransform()->Update()` 같은 호출을 통해 변환 행렬을 먼저 갱신해야 하는데, 현재는 테스트 용으로 주석처리된 상태이다.

```cpp
//GetTransform()->Update();

_material->Update();
_mesh->Render();
```

---

### Mesh 클래스

- 삭제
    - transform과 material 관련 기능들은 MeshRenderer에서 처리하도록 했고, Mesh는 순수하게 vertex와 index 데이터를 GPU에게 전달하는 역할만 한다.

```cpp
Transform _transform;
shared_ptr<Material> _mat;

void SetTransform(Transform transform);
void SetMaterial(shared_ptr<Material> mat);
```

---

### Game 클래스

- 헤더 추가
    - 렌더링에 필요한 GameObject와 MeshRenderer 클래스 사용을 위해 헤더를 포함한다.

```cpp
#include "GameObject.h"
#include "MeshRenderer.h"
```

&nbsp;

- `Game::Init()` 수정
    - Transform 컴포넌트를 자동으로 붙여준다.
    - Unity처럼 GameObject를 생성하면 자동으로 Transform이 포함되도록 한 것이다.
        ```cpp
        shared_ptr<GameObject> gameObject = make_shared<GameObject>();
        gameObject->Init(); // Transform 컴포넌트 추가
        ```
    - MeshRenderer 생성 및 설정
        - 새로운 MeshRenderer 컴포넌트를 생성하고, 이후 Mesh와 Material을 설정한다.
            ```cpp
            shared_ptr<MeshRenderer> meshRenderer = make_shared<MeshRenderer>();
            ```
    - Mesh 설정
        - vertex 정보 `vec`, index 정보 `indexVec을` 이용해 Mesh를 초기화하고, MeshRenderer에 연결한다.
            ```cpp
            shared_ptr<Mesh> mesh = make_shared<Mesh>();
            mesh->Init(vec, indexVec);
            meshRenderer->SetMesh(mesh);
            ```
    - Material 설정
        - 쉐이더와 텍스처를 초기화한다.
        - 쉐이더 파일과 텍스처 파일을 로딩한다.
            ```cpp
            shared_ptr<Shader> shader = make_shared<Shader>();
            shared_ptr<Texture> texture = make_shared<Texture>();
            shader->Init(L"..\\Resources\\Shader\\default.hlsli");
            texture->Init(L"..\\Resources\\Texture\\dog.jpg");
            ```
        - Material에 Shader와 Texture를 연결하고, float 값도 설정한다.
            ```cpp
            shared_ptr<Material> material = make_shared<Material>();
            material->SetShader(shader);
            material->SetFloat(0, 0.3f);
            material->SetFloat(1, 0.4f);
            material->SetFloat(2, 0.3f);
            material->SetTexture(0, texture);
            ```
        - MeshRenderer에 Material을 할당하여 이후 렌더링 시 사용되도록 한다.
            ```cpp
            meshRenderer->SetMaterial(material);
            ```
    - GameObject에 Component로 추가
        - MeshRenderer를 GameObject에 붙인다.
        - 내부에서 `SetObject(shared_from_this())`가 호출되어 GameObject와 Component간의 상호 참조가 연결된다.
            ```cpp
            gameObject->AddComponent(meshRenderer);
            ```

&nbsp;

- `Game::Update()`
    `gmaeObject->Update()`: 해당 Object의 모든 컴포넌트가 실행되도록 한다.

```cpp
GEngine->Update();

GEngine->RenderBegin();
gameObject->Update();
GEngine->RenderEnd();
```

&nbsp;

- 결과
    - 저번 수업에서 달라진 건 없지만, 컴포넌트를 사용해서 그리는 구조로 바뀌었다.

![ComponentResult](/assets/images/post_img/directx/ComponentResult)

---

[Component 커밋](https://github.com/chaeeun-dev/DirectX12/commit/e9ce5934d57c3ee725685aec6a2a6337eb559023)

---

## 마치며

- 이번 시간에 배운 것
    - GameObject에 여러 Component를 부착하는 구조 구현하기
    - 생명 주기 함수를 이용해 관리하는 방식 이해하기
    - `weak_ptr을` 사용해 순환 참조 방지, `shared_from_this()`로 자신을 안전하게 전달하기
    - MeshRenderer를 통해 Material과 Mesh를 연결하고 렌더링하기

&nbsp;

올인원 수업에서는 Unreal 방식으로 구현했는데 다렉은 Unity 방식으로 구현하니 다양하게 배워서 좋구만. 

이제 섹션 3이 거의 끝나간다.. 파이팅!

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*