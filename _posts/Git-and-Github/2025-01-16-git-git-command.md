---
title: "[Git] #6. 명령어로 깃 다루기"
excerpt: "명령어를 알아야 하는 이유와 명령어 사용하기"

categories:
  - Git
tags:
  - [Git, Command]

permalink: /theory/git/git-command

toc: true
toc_sticky: true

date: 2025-01-16
last_modified_at: 2024-01-16
---

## 깃 명령어를 알아야 하는 이유

소스트리로 commit, push, branch 등 공부 끝났는데, 굳이 왜 배워야 할까?
- 명령어를 잘 익혀두면 소스트리로 작업하는 것보다 훨씬 간결하고, 편리함.
- 소스트리를 사용할 수 없는 환경에서 사용할 수 있음.

앞의 이론을 잘 익혔다면 명령어 공부는 정말 빠르게 끝낼 수 있다!

- 운영 체제
    - Windows에서는 git bash
    - macOS에서는 터미널(terminal, iterm...)

---

## 명령어

### 저장소와 버전 만들기

기본적인 명령어
1. pwd: 현재 경로 확인하기
2. cd: 경로 이동하기
3. clear: 화면 깨끗하게 만들기

**[명령어]**
1. git init: 로컬 저장소 만들기
2. git status: 작업 디렉터리 상태 확인하기
3. git add: 스테이지에 올리기 (git add . : 모든 변경 사항을 스테이지에 올리기)
4. git commit: 커밋하기 (git commit -am : add와 commit을 한 번에 하기)
5. git log: 커밋 조회하기

- commit 메시지
    - `git commit -m "commit message"`
    - `git commit` 적고 Enter키 → VI 편집기에 a나 i 입력 후 `INSERT`뜨면 커밋 메시지 작성(제목, 엔터 후 본문) → `:wq`나 `:w`, `:q` 입력

---

### 커밋 다양하게 조회하기

1. git log --oneline: 한 줄에 log를 출력하기
2. git log -p or patch: 각 커밋의 변경 사항 출력하기
3. git log --graph: 그래프 보여주기 (한 줄로 그래프를 보고싶다면, git log --oneline --graph)

---

### 태그 관리하기

1. git tag 태그: 태그 추가하기 (git tag v0.0.1)
2. git tag --list: 태그 조회하기
3. git tag --delete 태그: 태그 삭제하기

---

### 작업 내용 비교하기

1. git diff / git cached: 최근 커밋과 작업 디렉터리 비교하기
2. git diff --staged: 최근 커밋과 스테이지 비교하기
3. git diff 커밋1 커밋2: 커밋끼리 비교하기 (순서 중요)
4. git diff 브랜치1 브랜치2: 브랜치끼리 비교하기

---

### 작업 되돌리기

1. git revert 커밋 해시: 버전을 되돌린 새로운 버전을 만들기
2. git reset 커밋 해시: 버전을 완전히 되돌리기
    - 디폴트 or git reset --mixed 커밋 해시: 스테이지까지 되돌리기
    - git reset --soft 커밋 해시: 커밋만 되돌리기
    - git reset --hard 커밋 해시: 커밋부터 스테이지, 작업 디렉터리까지 되돌림

---

### 작업 임시 저장하기

1. git stash: 임시 저장하기
2. git stash list: stash 목록 확인하기
3. git stash -m "Commit Message": stash 저장하기 (최근 항목이 0번이 됨)
4. git stash apply 적용할 스태시: 임시 저장된 항목을 적용하기
5. git stash drop 적용할 스태시: 임시 저장된 항목을 삭제하기

---

### 브랜치 관리하기

1. git branch: 현재 브랜치 보기 (e.x. 현재 mster 브랜치에 check out 되어 있으면, * master로 표시)
2. git branch 생성할 브랜치: 브랜치 생성하기 (check out은 X)
3. git checkout 체크아웃 할 브랜치: check out 하기
4. git checkout -b 생성할 브랜치: 브랜치를 생성함과 동시에 check out 하기
5. git log / git log --branches: 현재 브랜치의 기록 보기 / 모든 브랜치의 기록 보기
6. git merge 병합할 브랜치: 현재 브랜치에 병합하기

**[Merge] 예시**  
1. git checkout master: 병합 할 브랜치에 check out
2. git merge foo
3. git branch -d foo: 병합된 브랜치는 삭제하는 게 좋음

**[충돌 발생했을 때]**
1. 직접 수정(둘 중 하나만 남기기)
2. 커밋하기

---

### rebase 

**[rebase 예시]** 
1. git checkout foo: rebase 할 foo 브랜치로 check out 하기
2. git rebase master: master 브랜치로 rebase 함

---

### clone, push

1. git clone <SSH 주소>: SSH 주소로 원격 저장소 clone하기
2. cd <저장소 이름>: clone한 저장소로 이동하기
3. ls / ls -a: 저장소 내 파일 목록 확인하기 (-a 옵션은 숨김 파일까지 표시)
4. git branch -M main: master 브랜치 이름을 main으로 변경하기 (깃허브에서 하라는 대로 명령어 입력하면 됨)
5. git remote add origin <주소>: 원격 저장소 추가하기

- git remote -v: 등록된 원격 저장소 확인하기
- git push -u origin main: push하기 (최초 1회 실행하면 이후 git push만 입력해도 자동 푸시 가능)

---

### fetch, pull

- git fetch -u origin main: 원격 저장소에서 최신 변경 사항을 가져오지만, 자동 병합은 하지 않음 (최초 1회 실행하면 이후 git fetch만 입력해도 자동 푸시 가능)
- git pull -u origin main: 원격 저장소에서 최신 변경 사항 가져오기 (최초 1회 실행하면 이후 git pull만 입력해도 자동 푸시 가능)

---

### 풀 리퀘스트(pull request) 

- fork한 저장소가 원본 저장소보다 뒤쳐져있을 때 → 깃허브에서 Sync fork - Update branch 하기

**Pull Request**
1. 기여하려는 저장소를 본인 계정으로 fork하기
2. fork한 저장소를 clone하기
3. 브랜치 생성 후 생성한 브랜치에서 작업하기
4. 작업한 브랜치 push하기
5. pull request 보내기

**Pull Request 명령어**
1. git clone <SSH 주소>
2. cd <원격 저장소 이름>
3. git checkout -b <브랜치 이름>: 브랜치 생성과 동시에 check out (main 브랜치에 커밋하면 안 됨!)
4. 스테이지 추가 및 커밋
5. git push origin <브랜치 이름>: 명시적으로 브랜치 이름을 입력해야 함
6. 깃허브에서 내용 확인 후 pull request 보냄
7. 원래 저장소의 주인이 확인 후 merge 하면 끝

---

## 연습하기

커밋 계속 만들면서 연습하기 귀찮아서 gpt한테 문제 내달라고 했다.. 이렇게 하니까 훨씬 재밌다 ㅎㅎ 혼자 레벨도 높여준다..

![Image](https://github.com/user-attachments/assets/2231d777-fcc1-49b3-abd4-01460ccced86)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*