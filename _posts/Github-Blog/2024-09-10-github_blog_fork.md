---
title: "[Github Blog] Jekyll 블로그 잔디 안 심어지는 문제 해결"
excerpt: "깃허브 블로그 잔디 안 심어지는 문제 해결하기"

categories:
  - Github Blog
tags:
  - [Github Blog, 잔디 심기]

permalink: /projects/github-blog/grass-error

toc: true
toc_sticky: true

date: 2024-09-09
last_modified_at: 2024-09-09
---

## 문제

깃허브 블로그에 매일 til을 작성하면서 1일 1잔디를 심으려고 했지만, 깃허브 블로그에 커밋을 하면 잔디가 안 심어지는 문제가 발생했다(다른 레파지토리에서 커밋하면 정상적으로 반영 됨). 

메일과 아이디 문제는 아니었고 fork 했을 때 branch 설정 때문인 것 같았다.

깃 사용 방법을 잘 몰라서 타 블로그를 봐도 잘 이해하지 못했지만 깃허브의 copilot 도움으로 문제를 해결했다.

---

## 참고한 깃허브 코파일럿 답변

1. 터미널을 엽니다.
2. 포크한 저장소의 'bare' 클론을 생성합니다:
    
    ```bash
    bash
    코드 복사
    git clone --bare https://github.com/EXAMPLE-USER/FORK-NAME.git
    
    ```
    
    여기서 `EXAMPLE-USER`와 `FORK-NAME`은 각각 자신의 GitHub 사용자명과 포크한 저장소의 이름으로 대체해야 합니다.
    
3. 포크된 저장소를 삭제합니다. 주의할 점은 이 작업을 실행하면 연관된 모든 풀 리퀘스트 및 설정이 영구적으로 삭제되며, 이 작업은 되돌릴 수 없습니다. 자세한 내용은 "저장소 삭제" 문서를 참조하세요.
4. 동일한 위치에 동일한 이름의 새 저장소를 생성합니다. 자세한 내용은 "새 저장소 만들기" 문서를 참조하세요.
5. 저장소를 동일한 원격 URL로 mirror-push 합니다:
    
    ```bash
    bash
    코드 복사
    cd FORK-NAME.git
    git push --mirror https://github.com/EXAMPLE-USER/FORK-NAME.git
    
    ```
    
    다시 한 번, `EXAMPLE-USER`와 `FORK-NAME`은 각각 자신의 GitHub 사용자명과 저장소 이름으로 대체해야 합니다.
    
6. 이전에 생성한 임시 로컬 클론을 삭제합니다:
    
    ```bash
    bash
    코드 복사
    cd ..
    rm -rf FORK-NAME.git
    
    ```
    

이 작업을 완료하면 포크한 저장소가 독립적인 새로운 저장소로 전환되며, 이로 인해 해당 커밋들이 기여 내역에 포함될 수 있습니다.

---

## 해결 
나는 이전 블로그 데이터를 copy한 후 repository를 삭제하는 대신 이름을 바꿨다. 그리고 새로 생성한 repository 이름으로 블로그 사이트에 들어갔는데 404 표시가 뜨길래 10분 이상 기다렸지만 문제가 해결되지 않았다. 
그래서 아무 내용을 집어 넣고 커밋 해보니 정상적으로 사이트가 표시 됐다!
