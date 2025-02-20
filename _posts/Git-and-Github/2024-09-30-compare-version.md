---
title: "[Git] #3-1. 버전 비교하기"
excerpt: "이전 버전과 최근 버전 비교하기 실습"

categories:
  - Git
tags:
  - [Git, CompareVersion]

permalink: /theory/git/compare-version

toc: true
toc_sticky: true

date: 2024-09-30
last_modified_at: 2024-09-30
---

## 버전 비교하기

직전 버전과 비교해보자. 추가 됐으면 +표시와 초록색이, 삭제 됐으면 -표시와 빨간색이 표시된다.

![image](https://github.com/user-attachments/assets/f0828565-e64a-4b21-ae99-732db2bc903a) | ![image](https://github.com/user-attachments/assets/fe4c43ca-f460-41ce-b400-6ab5ba553466)


버전 별로 비교하고 싶을 때(예를 들어 두 번째 커밋과 다섯 번째 커밋의 차이를 비교)를 알아보자.

일단 두 번째 커밋 당시의 a.txt 파일을 확인할 수 있다. 두 번째 커밋을 클릭하고 a.txt를 우클릭 후 **선택한 버전 열기**를 클릭한다. 그러면 그 당시의 파일 내용을 확인할 수 있다. 

![image](https://github.com/user-attachments/assets/3bb98d2f-779a-4249-a1a1-0d962966289f)


그러면 두 번째 버전에서 다섯 번째 버전을 비교했을 때 달라진 점을 확인하는 방법을 알아보자. ctrl을 누르고 두 번째 버전과 다섯 번째 버전을 클릭한 후, 비교할 파일인 a.txt를 선택하면 우측 하단에 버전 차이를 확인할 수 있다.

![image](https://github.com/user-attachments/assets/195bb269-272d-4fe3-b756-0dfe57e559e5)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*