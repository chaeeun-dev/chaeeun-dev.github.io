---
title: "[PROJECT] 개발 고민 모음"
date: 2024-12-09 12:50:00 +09:00 
categories: PROJECT
author_profile: false
toc : true
toc_sticky: true
---

고민 적는 방법
- 생각
- 구현 방법, 과정
- 발생한 오류

### Tilemap 충돌 구현(24.12.09)
- 생각 - player와 tilemap의 충돌을 처리하기 위해 tilemap에 collider를 붙여야겠다고 생각했다.
- 구현 -  DevScene의 Init에서 Tilemap을 Load한 후 tiles[y][x]의 모든 정보를 불러와 '1'이면(타일이면) BoxCollider를 그 타일 위치에 생성하고자 했다. 
- 오류 - Vector subscript out of range 오류가 발생했다. 
    - 타일이 너무 많아 vector resize를 안 해서 오류가 발생하는 줄 알았다. Actor 클래스의 ```_conponents```와 별개로 ```_tileComponents``` 변수를 만들어 resize를 충분히 했는데도 해결되지 않았다.
    - for문의 ```<=```를 ```<```로 바꿔서 해결이 됐다. 이 오류는 주로 인덱스가 벡터의 유효 범위를 벗어날 때 벗어난다고 한다. 

![image](https://github.com/user-attachments/assets/a07882f0-5acf-4feb-a7ff-315a5b509a74)

- 문제 - 오류가 해결 되었는데, 문제는 렉이 너무 많아 걸렸다. 1500개가 넘는 BoxCollider를 생성하느라 메모리, 시간적인 측면에서 버거웠던 것 같다. 

```cpp
// Tilemap Collision
{
	// Tilemap Load
	GET_SINGLE(ResourceManager)->LoadTilemap(L"Tilemap", L"Tilemap\\Tilemap.txt");	// x : 160개, y : 36개 
	Tilemap* tm = GET_SINGLE(ResourceManager)->GetTilemap(L"Tilemap");

	{
		Vec2Int mapSize = tm->GetMapSize();

		std::vector<std::vector<Tile>>& tiles = tm->GetTiles();
		Vec2Int winSize = GET_SINGLE(ValueManager)->GetWinSize();

		for (int32 y = 0; y < mapSize.y; ++y)
		{
			for (int32 x = 0; x < mapSize.x; ++x)
			{
				if (tiles[y][x].value == 1)
				{
					Actor* tile = new Actor();
					tile->SetPos({ (float)x * MAP_TILE_SIZEX, (float)y * MAP_TILE_SIZEY });
					tile->SetLayer(LAYER_TILEMAP);

					{
						BoxCollider* collider = new BoxCollider();
						collider->SetSize({ 24, 24 });
						collider->SetCollisionLayer(CLT_GROUND);
						GET_SINGLE(CollisionManager)->AddCollider(collider);
						tile->TileAddComponent(collider);
					}
				}
			}
		}
	}
}
```

- 해결 - 직접 위치에 따라 BoxCollider를 생성하는 방식으로 구현
- 고민 - 근데 '이게 맞나..?'라는 생각이 든다. DevScene의 Init 부분에서 Actor를 생성하고, 위치를 직접 작성하여 BoxCollider를 추가하는 방식으로 했는데 코드가 너~무 길어졌다.. 근데 Tilemap.txt를 읽어서 덩어리를 만드려면 구현이 굉장히 어렵고 오래걸릴 것이다(인접한 1들의 외각선 박스를 만드는 알고리즘을 구현해야 해서 아마 몇 주 이상은 걸릴 것 같다). 일단은 이 방법으로 가야겠다..! 