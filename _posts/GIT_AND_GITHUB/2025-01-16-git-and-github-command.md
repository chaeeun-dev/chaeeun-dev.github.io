---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션6 명령어로 깃 다루기"
date: 2025-01-16 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

## [이론] 깃 명령어를 알아야 하는 이유
소스트리로 commit, push, branch 등 공부 끝났는데, 굳이 왜?
- 명령어를 잘 익혀두면 소스트리로 작업하는 것보다 훨씬 간결하고, 편리함.
- 소스트리를 사용할 수 없는 환경에서 사용할 수 있음.

앞의 이론을 잘 익혔다면 명령어 공부는 정말 빠르게 끝낼 수 있다!

- 운영 체제
    - Windows에서는 git bash
    - macOS에서는 터미널(terminal, iterm...)

---

## [Windows 실습] 명령어

### 저장소와 버전 만들기
기본적인 명령어
1. pwd : 현재 경로 확인하기
2. cd : 경로 이동하기
3. clear : 화면 깨끗하게 만들기

명령어
1. git init : 로컬 저장소 만들기
2. git status : 작업 디렉터리 상태 확인하기
3. git add : 스테이지에 올리기 (git add . : 모든 변경 사항을 스테이지에 올리기)
4. git commit : 커밋하기 (git commit -am : add와 commit을 한 번에 하기)
5. git log : 커밋 조회하기

- commit 메시지
    - `git commit "commit message"`
    - `git commit` 적고 Enter키 → VI 편집기에 a나 i 입력 후 `INSERT`뜨면 커밋 메시지 작성(제목, 엔터 후 본문) → `:wq`나 `:w`, `:q` 입력

---

### 커밋 다양하게 조회하기
1. git log --oneline : 한 줄에 log를 출력하기
2. git log -p or patch : 각 커밋의 변경 사항 출력하기
3. git log --graph : 그래프 보여주기 (한 줄로 그래프를 보고싶다면, git log --oneline --graph)

---

### 태그 관리하기
1. git tag 태그 : 태그 추가하기 (git tag v0.0.1)
2. git tag --list : 태그 조회하기
3. git tag --delete 태그 : 태그 삭제하기

---

## 작업 내용 비교하기
1. git diff / git cached : 최근 커밋과 작업 디렉터리 비교하기
2. git diff --staged : 최근 커밋과 스테이지 비교하기
3. git diff 커밋1 커밋2 : 커밋끼리 비교하기 (순서 중요)
4. git diff 브랜치1 브랜치2 : 브랜치끼리 비교하기

---