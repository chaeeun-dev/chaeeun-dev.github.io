---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션3 버전 가지고 놀기"
date: 2024-09-30 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

# 버전 가지고 놀기


## [Windows 실습] 버전 비교하기(24.09.30)

직전 버전과 비교해보자. 추가 됐으면 +표시와 초록색이, 삭제 됐으면 -표시와 빨간색이 표시된다.

![image](https://github.com/user-attachments/assets/f0828565-e64a-4b21-ae99-732db2bc903a) | ![image](https://github.com/user-attachments/assets/fe4c43ca-f460-41ce-b400-6ab5ba553466)


버전 별로 비교하고 싶을 때(예를 들어 두 번째 커밋과 다섯 번째 커밋의 차이를 비교)를 알아보자.

일단 두 번째 커밋 당시의 a.txt 파일을 확인할 수 있다. 두 번째 커밋을 클릭하고 a.txt를 우클릭 후 **선택한 버전 열기**를 클릭한다. 그러면 그 당시의 파일 내용을 확인할 수 있다. 

![image](https://github.com/user-attachments/assets/3bb98d2f-779a-4249-a1a1-0d962966289f)


그러면 두 번째 버전에서 다섯 번째 버전을 비교했을 때 달라진 점을 확인하는 방법을 알아보자. ctrl을 누르고 두 번째 버전과 다섯 번째 버전을 클릭한 후, 비교할 파일인 a.txt를 선택하면 우측 하단에 버전 차이를 확인할 수 있다.

![image](https://github.com/user-attachments/assets/195bb269-272d-4fe3-b756-0dfe57e559e5)



## [이론] 버전을 되돌리는 방법(24.10.01) 

만들어진 버전을 되돌리는 두 가지 방법
1. revert - 버전을 되돌린 새로운 버전 만들기
2. reset - 버전을 완전히 되돌리기

**revert** - 지금까지 만든 모든 버전이 유지된 채 새로운(이전 버전의 복제(?)) 버전을 만든다. **기존의 버전은 삭제되지 않는다.**

![image](https://github.com/user-attachments/assets/fca18fcb-d404-420f-8d89-a3e7977fa2e7)


**reset** - 되돌아갈 버전의 시점으로 완전하게 되돌아가는 방식이다. 즉, 되돌아갈 버전 이후의 버전은 삭제되는 방식이다.

하나의 버전이 만들어지는 과정 중
1. ~~작업 디렉터리에서 변경 사항 생성하기~~ → hard reset(작업 디렉터리까지 되돌리기)
2. ~~스테이지로 추가하기~~ → mixed reset(스테이지까지 되돌리기) 
3. ~~저장소로 커밋하기~~ → soft reset(커밋만 되돌리기)


---

출처 [인프런 강민철님 모두의 깃&깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)