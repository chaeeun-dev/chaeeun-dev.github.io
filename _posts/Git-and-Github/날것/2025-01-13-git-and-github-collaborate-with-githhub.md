---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션5 깃허브로 협업하기"
date: 2025-01-13 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

## [Windows 실습] 원격 저장소와의 네 가지 상호작용
- 원격 저장소와의 네 가지 상호작용
    - clone : 원격 저장소를 복제하기
    - push : 원격 저장소에 밀어넣기
    - fetch : 원격 저장소를 일단 가져만 오기
    - pull : 원격 저장소를 가져와서 합치기

---

### clone
(깃허브 상에 존재하는) 원격 저장소를 로컬(클론받은 컴퓨터로) 복제

원격 저장소 브랜치 이름
- main branch == master branch
- origin == 원격 저장소에 붙은 일종의 별명
- origin/HEAD == 원격 저장소 origin의 HEAD
- origin/main == 원격 저장소 origin의 main

### push
원격 저장소에 로컬 저장소의 변경 사항을 밀어넣기

### fetch
원격 저장소를 일단 가져만 오는 것, 로컬 브랜치와 병합되지 않음.

### pull
원격 저장소를 가져와서 합치기

---

## [Windows 실습] 풀 리퀘스트 : 깃허브로 협업하기
일반적으로 내가 소유하지 않은 원격 저장소에 푸쉬할 수 없다.

### Pull Request
원격 저장소가 내 변경사항을 Pull하도록 Request를 보내는 방식

1. 기여하려는 저장소를 본인 계정으로 fork하기
2. fork한 저장소를 clone하기
3. branch 생성 후, 생성한 브랜치에서 작업하기
4. 작업한 branch push하기
5. pull request 보내기