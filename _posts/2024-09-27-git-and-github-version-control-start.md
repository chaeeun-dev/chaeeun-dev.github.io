---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션2 깃으로 버전 관리 시작하기"
date: 2024-09-26 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

# 깃으로 버전 관리 시작하기

## [Windows 실습] 로컬 저장소 만들기(24.09.27)

저장소 : 깃으로 버전을 관리할 수 있는 공간

source tree를 실행해서 저장소를 만든다.

![image](https://github.com/user-attachments/assets/08c08819-74d0-4f7e-8efb-6c9f8fd717b6)

.git 파일이 안 보여서 파일 옵션에서 설정 변경했더니 .git 파일이 보인다.

![image](https://github.com/user-attachments/assets/8c245dc3-e3cd-426d-ba60-b3b557b56a04)

![image](https://github.com/user-attachments/assets/7a04a4c3-9836-49d2-bba2-d53b7615bfe5)



## [이론] 버전 관리의 큰 그림(24.09.27)

**깃이 관리하는 세 개의 공간** 

![image](https://github.com/user-attachments/assets/71d6ad1d-42a4-46e8-81cb-a1a159fdf20e)

- 작업 디렉터리 - 버전 관리의 대상이 위치하는 공간(.git이 있는 디렉터리)
- 스테이지(인덱스INDEX) - 다음 버전이 될 후보가 올라가는 공간
- 저장소 - 버전이 만들어지고 관리되는 공간

스테이지와 저장소는 깃이 관리하는 가상의 공간(보이지 않는 곳)

예를 들어 작업 디렉터리에 파일 1000개가 있고 이 중에서 100가 수정되거나 삭제 됐을 때 모든 변경 사항이 새로운 버전이 되어야 하는 건 아니다. 새 버전으로 만들만큼 중요하지 않거나, 임시로 변경했거나, 실수로 변경했을 경우 굳이 새로운 버전으로 만들 필요는 없다. → **변경 사항이 생겼다고 해서 무조건 새로운 버전으로 만들 필요는 없다**

100개 중 새로운 버전이 될 파일을 선별하는 작업이 필요하다. 새로운 버전이 될 파일만 특별한 공간, 스테이지(스테이징 영역staging area, 인덱스index)로 옮기는 작업을 거친다. 스테이지는 변경 사항이 있는 파일 중 다음 버전이 될 후보가 올라가는 공간이다. 

![image](https://github.com/user-attachments/assets/87ed8549-1f4c-4d16-9958-f38a6ebf5aee)

스테이지에 있는 파일을 바탕으로 새로운 버전을 만들면 새 버전이 저장소에 추가된다. 작업 디렉터리에서 만들어진 모든 버전들의 내역이 저장소에 있다. → **저장소는 버전이 만들어지고 관리되는 공간이다**

![image](https://github.com/user-attachments/assets/70dcffcd-88eb-4e46-b759-96dfb6a997ad)

이러한 과정을 반복하며 저장소에는 새로운 버전들이 차곡차곡 쌓이게 된다. 

작업 디렉터리에서 버전이 될 후보 파일을 스테이지에 옮기는 것을 ‘스테이지에 추가한다(add)’, ‘해당 파일을 스테이지시킨다(staged)’라고 한다. 그리고 스테이지에 추가된 파일을 ‘추가된(add) 파일’, ‘스테이지된(staged) 파일’이라고 표현한다.  

![image](https://github.com/user-attachments/assets/1dcb8ef0-b10e-419c-ab27-23bfcc432a82)


저장소에 새로운 버전을 만드는 것을 ‘커밋한다(commit)’라고 한다. 저장소에 저장된 각각의 버전들을 커밋이라 부르기도 한다. ‘깃으로 만든 버전’을 편의상 ‘커밋’이라고 지칭할 것!

![image](https://github.com/user-attachments/assets/55215a5e-6de4-41e0-85cb-8ccb962f0335)

**정리** - 작업 디렉터리의 파일은 변경 사항 생성 → add → commit 과정 통해 작업 디렉터리 → 스테이지 → 저장소 순으로 이동하며 새로운 버전으로 만들어진다.


---

출처 [인프런 강민철님 모두의 깃&깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)