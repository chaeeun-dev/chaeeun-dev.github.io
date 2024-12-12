---
title: "[ERROR] SERVER 오류 모음"
date: 2024-08-04 05:20:00 +09:00 
categories: ERROR
author_profile: false
toc : true
toc_sticky: true
---

### 모호한 기호 (24.12.12)
- 오류 : 서버 코드에서 'byte가 모호합니다' 라는 오류가 발생했다. 

![image](https://github.com/user-attachments/assets/a4cd7303-2ed6-485f-8e83-b425e55b78cc)

- 해결 과정 : [모호한 기호입니다](https://swengineer.tistory.com/44) 이 글을 보고 오류 원인을 알아냈다. **C++ 17에서 추가된 std::byte와 windows.h의 byte 정의가 중복되어서 발생한 것이다.** 
- 해결 방법 : 프로젝트 속성 - C/C++ 전처리기 정의에 `HAS_STD_BYTE=0`을 추가하면 해결된다. 여기서 띄어쓰기 하면 안 됨. 또는 pch에 `define _HAS_STD_BYTE 0`를 사용해도 된다. (사실 `using namespace std;`를 쓰지 않고 표준 함수에 `std::`를 붙이면 바로 해결되는 문제다! std::byte 이 형식으로 쓰면 된다.)
- 배움 : `using namespace std;` 쓰지 말고 표준 함수에 `std::`을 붙이자. (강의 들을 땐 코드 받아서 사용하니까 어쩔 수 없었다. 프로젝트 진행할 때는 `std::`로 쓰고 있으니까 앞으로도 쭉 이렇게 해야겠다.) `using namespace std;`를 써도 문제가 발생한 적이 없어서 쓰지 말라는 이유를 몰랐는데 이제야 깨닫게 되었다. 겹치는 경우가 존재하구나..! 저거 때문에 오류가 생기는구나..!!