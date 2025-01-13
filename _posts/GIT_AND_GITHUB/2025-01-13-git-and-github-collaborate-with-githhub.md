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