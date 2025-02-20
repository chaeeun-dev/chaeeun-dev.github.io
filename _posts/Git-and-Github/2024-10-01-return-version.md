---
title: "[Git] #3-2. 버전을 되돌리는 방법"
excerpt: "버전 되돌리는 방법 revert, reset 이론"

categories:
  - Git
tags:
  - [Git, Version]

permalink: /theory/git/return-version

toc: true
toc_sticky: true

date: 2024-10-01
last_modified_at: 2024-10-01
---

## [이론] 버전을 되돌리는 방법(24.10.01) 

만들어진 버전을 되돌리는 두 가지 방법
1. revert - 버전을 되돌린 새로운 버전 만들기
2. reset - 버전을 완전히 되돌리기

### revert
지금까지 만든 모든 버전이 유지된 채 새로운(이전 버전의 복제(?)) 버전을 만든다. **기존의 버전은 삭제되지 않는다.**

![image](https://github.com/user-attachments/assets/fca18fcb-d404-420f-8d89-a3e7977fa2e7)


### reset
되돌아갈 버전의 시점으로 완전하게 되돌아가는 방식이다. 즉, 되돌아갈 버전 이후의 버전은 삭제되는 방식이다.

하나의 버전이 만들어지는 과정 중
1. ~~작업 디렉터리에서 변경 사항 생성하기~~ → hard reset(작업 디렉터리까지 되돌리기)
2. ~~스테이지로 추가하기~~ → mixed reset(스테이지까지 되돌리기) 
3. ~~저장소로 커밋하기~~ → soft reset(커밋만 되돌리기)

---

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*