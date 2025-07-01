---
title: "[2DGP] 생명 주기"
excerpt: ""

categories:
  - 2DGame Project
tags:
  - [DirectX]

permalink: /2dgame-project/life-cycle/

toc: true
toc_sticky: true

date: 2025-07-01
last_modified_at: 2025-07-01
---

## 생명 주기

### 작동 문제

GameScene에서 스테이지1을 생성하고 플레이할 땐 문제가 없는데, 스테이지 전환 시 기존 Scene에 남아있는 Actor들을 지우고 스테이지2의 Actor들을 새로 생성했는데, 타일맵과 몬스터가 제대로 생성되지 않는 문제가 발생했다. 

- 구조
    - GameObject와 Creatrue 등의 최상위 부모 Actor는 다음과 같은 생명 주기 함수를 가진다.
        ```cpp
        public:
            Actor();
            virtual ~Actor();

            virtual void BeginPlay();	// Init
            virtual void Tick();		// Update
            virtual void Render(HDC hdc);
        ```
    - Scene 클래스의 `Init()`에서 등록된 Actor들의 `BeginPlay()`를 호출한다. 즉, `Scene::Init()`을 호출해야 Actor가 본격적으로 시작되는 것!
        ```cpp
        void Scene::Init()
        {
            for (const std::vector<Actor*>& actors : _actors)
                for (Actor* actor : actors)
                    actor->BeginPlay();

            for (Panel* panel : _panels)
                panel->BeginPlay();
        }
        ```

---

### 해결

- 문제
    - 게임 중 Stage를 전환하면서,
    1. 이전 Stage의 Actor들을 삭제
    2. 새로운 Actor를 생성했
    3. Scene은 그대로 둠(`Scene::Init()`을 호출하지 않음) 
    - 그 결과 Actor들의 `BeginPlay()`가 호출되지 않아, `Tick()`과 `Render()`가 동작하지 않았다. (2시간 정도 삽질함)
- 해결
    - 스테이지를 전환하는 SetStage 함수에 Actor를 생성한 다음 `Super::Init()`이 호출되도록 코드를 추가한다.
        ```cpp
        void GameScene::SetStage(int stage)
        {
            // 1. 이전 액터 삭제
            // 2. 새로운 액터 생성
            // ...

            // 3. BeginPlay가 다시 호출되도록
            Super::Init(); // Scene::Init()
        }
        ```
    - Actor들의 BeginPlay()가 정상적으로 호출되며 문제가 해결됐다!

---

## 마치며

`Super::Init()` 한 줄이 모든 걸 해결해줬다니.. 당연히 `BeginPlay()`가 문제라고 생각하지 않아서 더 오래걸린 것 같다. 뭔가 문제가 생기면 당연한 걸 빼먹은 게 아닌지 고민해보는 습관을 가져야겠다.