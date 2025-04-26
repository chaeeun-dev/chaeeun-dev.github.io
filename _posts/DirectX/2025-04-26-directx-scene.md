---
title: "[DirectX 12] #3-4. Scene"
excerpt: "Scene 개념에 대한 이해와 Manager 구현"

categories:
  - DirectX12
tags:
  - [DirectX, Scene, Manager]

permalink: /DirectX12/scene/

toc: true
toc_sticky: true

date: 2025-04-26
last_modified_at: 2025-04-26
---

## 들어가며

이전까지는 게임의 객체를 Client의 Game 클래스에서 관리했는데, 이제는 Scene에서 관리하는 방식으로 바꿔본다. 이번 시간에는 Scene에 대해 알아보고 Scene과 SceneManager를 구현해본다.

---

## Scene

- Scene이란?
    - 여러 GameObject를 보관하고, 생명 주기에 따라 각각의 객체들을 일괄적으로 관리한다.
    - 예를 들어, 게임을 시작하면 로그인 Scene, 로그인 Scene, 게임 Scene처럼 각각의 상태나 공간을 Scene으로 나누어 관리한다.
    - 각 Scene은 자신만의 GameObject들을 가지고 있고, 현재 활성화된 Scene만 업데이트된다. 필요시 새로운 Scene으로 교체되는데, 이 때 이전의 Scene의 객체들의 메모리를 정리하는 작업이 필요하다.

&nbsp;

- 멤버 변수
    - scene에서 관리하는 모든 Object를 보관하는 벡터이다.

```cpp
vector<shared_ptr<GameObject>> _gameObjects;
```

&nbsp;

- 객체 추가/삭제
    - `Scene::AddGameObject()`
        ```cpp
	    _gameObjects.push_back(gameObject);
        ```
    - `Scene::RemoveOBject()`
        ```cpp
        auto findIt = std::find(_gameObjects.begin(), _gameObjects.end(), gameObject);
        if (findIt != _gameObjects.end())
        {
            _gameObjects.erase(findIt);
        }
        ```

&nbsp;

- 생명 주기 함수
    - 참고. 스마트 포인터를 `const shared_ptr<>&` 형태로 참조하는 이유는 reference count 증가를 막기 위함이다. (참조 형태로 사용하지 않으면 reference count가 증가됨)

```cpp
void Scene::Awake()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
		gameObject->Awake();
}

void Scene::Start()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
		gameObject->Start();
}

void Scene::Update()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
		gameObject->Update();
}

void Scene::LateUpdate()
{
	for (const shared_ptr<GameObject>& gameObject : _gameObjects)
		gameObject->LateUpdate();
}
```

---

## SceneManager 클래스

현재 활성화된 Scene을 관리하는 매니저로, 싱글톤 패턴으로 구현되어 전역에서 하나의 인스턴스만 존재한다.

&nbsp;

- 싱글톤 매크로(`EngniePch.h`)
    - `DECLARE_SINGLE(type)` 매크로를 클래스에 선언하면, 해당 클래스는 전역에서 하나의 인스턴스만 가질 수 있다.
    - `GET_SINGLE(type)` 매크로를 통해 인스턴스를 어디서든 가져올 수 있다.

```cpp
#define DECLARE_SINGLE(type) \
private: \
	type() {} \
	~type() {} \
public: \
	static type* GetInstance() \
	{ \
		static type instance; \
		return &instance; \
	}

#define GET_SINGLE(type)	type::GetInstance()
```

&nbsp;

- 멤버 변수
    - 현재 실행 중인 Scene을 저장한다.

```cpp
shared_ptr<Scene> _activeScene;
```

&nbsp;

- `SceneManager::Update()`
    - 현재 활성화된 Scene이 존재하면, 해당 Scene의 `Update()`와 `LateUpdate()`를 호출한다.

```cpp
if (_activeScene == nullptr)
    return;

_activeScene->Update();
_activeScene->LateUpdate();
```

&nbsp;

- `SceneManager::LoadScene()`
    - 기존 Scene을 정리하고 새로운 Scene을 load한다.
    - `Awake()` → `Start()` 호출로 초기화를 진행한다.

```cpp
void SceneManager::LoadScene(const wstring& sceneName)
{
	// TODO: 기존 Scene 정리
	// TODO: 파일에서 Scene 정보 로드

	_activeScene = LoadTestScene();

	_activeScene->Awake();
	_activeScene->Start();
}
```

&nbsp;

- `SceneManager::LoadTestScene()`
    - 테스트 전용 Scene을 임시로 구성하고 반환한다.
    - 이전에 Client에서 구현했던 GameObject 관련 코드를 옮겨 구성한다.

```cpp
	shared_ptr<Scene> scene = make_shared<Scene>();

	// GameObject 생성 및 구성 (Client의 Game::Init()에서 가져온 코드)
	// ...

	scene->AddGameObject(gameObject);
	return scene;
```

---

### Input, Timer 클래스 

전역으로 관리되던 Input과 Timer 클래스를 SceneManager와 마찬가지로 싱글톤으로 수정해보자.

&nbsp;

- 클래스 선언부 수정

```cpp
DECLARE_SINGLE(Input);
DECLARE_SINGLE(Timer);
```

&nbsp;

- 매크로 수정(`EnginePch.h`)

```cpp
#define INPUT		GET_SINGLE(Input)
#define DELTA_TIME	GET_SINGLE(Timer)
```

&nbsp;

- Engine 클래스 수정
    - `Engine::Init()`
        ```cpp
        GET_SINGLE(Input)->Init(info.hwnd);
        GET_SINGLE(Timer)->Init();
        ```
    - `Engine::Update()`
        ```cpp
        GET_SINGLE(Input)->Update();
        GET_SINGLE(Timer)->Update();

        Render();
        ```
    - `Engine::Render()`
        ```cpp
        RenderBegin();
        GET_SINGLE(SceneManager)->Update(); // 여기서 Scene의 Update 호출됨
        RenderEnd();
        ```

---

## Game 클래스

Game 클래스에서는 Scene의 load와 Engine의 Update만 실행되도록하고, GameObject는 Scene에서 관리하도록 수정해보자.

&nbsp;

- `Game::Init()` 수정
    - SceneManager에서 알아서 GameObject를 초기화한다.

```cpp
GEngine->Init(info);

GET_SINGLE(SceneManager)->LoadScene(L"TestScene"); 
```

&nbsp;

- `Game::Update()` 수정
    - Scene의 update 흐름이 포함된다. 
```cpp
GEngine->Update(); 
```

&nbsp;

- 결과
    - Client(Game 클래스)에서는 Scene 로드만 수행하고, Engine(Scene 클래스)에서는 GameObject를 관리하도록 나눴다.

---

## 마치며

- 이번 시간에 배운 것
    - Scene에 대한 개념과 클래스로 구현하기
    - 싱글톤으로 SceneManager, Input, Timer 구현하기
    - Engine과 Client에서 관리할 작업 나누기

섹션3이 드디어 끝났다!!!!! 섹션 4도 달려라.

---

*출처* 
*[인프런 Rookiss님 게임 수학과 DirectX12](https://www.inflearn.com/course/%EC%96%B8%EB%A6%AC%EC%96%BC-3d-mmorpg-2/dashboard)*