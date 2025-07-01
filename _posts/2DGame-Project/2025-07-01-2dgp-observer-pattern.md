---
title: "[2DGP] Observer 패턴"
excerpt: "불필요한 Update를 방지하는 관찰자 패턴"

categories:
  - 2DGame Project
tags:
  - [2DGP]

permalink: /2dgame-project/observer-pattern/

toc: true
toc_sticky: true

date: 2025-07-01
last_modified_at: 2025-07-01
---

## 관찰자 패턴

### HP와 UI

InGameUI 클래스에서 Player hp를 가져와 체력바를 그리려고 한다. 

- Player의 체력은 몬스터 공격, 힐템 등의 특정 이벤트에만 변경되는데, UI는 매 프레임마다 Player의 체력을 확인할 필요가 없다.
- 그래서 관찰자 패턴을 사용하여 InGamePanel이 Player의 체력 변화를 **구독**하고, 체력이 바뀔 때만 알림을 받아 업데이트 할 수 있도록 한다. 

> 💡 관찰자 패턴(Observer Pattern)이란?
>
> 어떤 객체의 상태가 변경될 때, 이를 구독하고 있는 객체들에게 자동으로 알림을 보내는 디자인 패턴이다. 
>
> 이 패턴을 활용하면 UI가 Player의 상태를 직접 체크하는 게 아니라, 변화가 있을 때만 알림을 받는 구조로 만들어 성능을 최적화 할 수 있다.

---

### 구현

1. Player 클래스에 Observer를 등록한다.
    - Observer 함수를 등록해 체력 변화 이벤트를 구독한다.
        ```cpp
    	std::function<void(int)> _healthObserver;	// 체력 변화 알림 받을 함수 포인터

        using HealthObserver = void(*)(int); // 콜백 함수 타입 정의

        void SetHealthObserver(std::function<void(int)> observer) {
            _healthObserver = observer; // 옵저버 함수 저장
        }
        ```
2. GameScene(`Init()`)에서 InGameUI를 생성하고, healthObserver를 등록한다.
    - InGamePanel을 생성하고, Player 객체에 설정한 다음, 채력 변화 시 호출될 람다 함수를 Observer로 등록한다.
    - 이 람다는 `UpdateHealthPoint`를 호출하여 체력바를 갱신한다.
        ```cpp
        // InGame UI 생성
        InGamePanel* panel = new InGamePanel();
        panel->SetPlayer(_player);
        AddPanel(panel);

        // 체력 변경 시 UI 업데이트 등록
        _player->SetHealthObserver([panel](int health) {
            if (panel)
                panel->UpdateHealthPoint(health);
        });
        ```
3. Player의 체력이 변경되면 UI에 알림을 전송한다.
    - 체력 변경 시(직접 설정, 증가/감소) `_healthObserver`가 호출되어 UI에 체력 변화가 전달된다.
        ```cpp
        void Player::SetHealthPoint(int hp)
        {
            _playerStat->hp = hp;

            // 관찰자에게 알림
            _healthObserver(_playerStat->hp);
        }

        void Player::AddHealthPoint(int hp)
        {
            // 더하는 로직
            // ...

            _healthObserver(_playerStat->hp);
        }

        void Player::SubtractHealthPoint(int hp)
        {
            // 빼는 로직
            // ...

            _healthObserver(_playerStat->hp);
        }
        ```
4. InGameUI에서 체력값을 수신하고, 값을 갱신한다.
    - Observer로 등록된 콜백 내에서 `InGameUI::UpdateHealthPoint()`가 호출된다.
    - 변경된 값으로 Render된다.
        ```cpp
        void InGamePanel::UpdateHealthPoint(int32 health) {
            SetHealthPoint(health); // 체력바 갱신
        }
        ```

---

## 마치며

2D 프로젝트에 대해 고민했던 것들을 포스팅하려고 했는데 거의 다 끝나갈 때 쓰게 됐다..ㅎㅎ 그래도 노트에 적어놔서 다행이다. 남은 것들 정리하면서 복습해야겠다!