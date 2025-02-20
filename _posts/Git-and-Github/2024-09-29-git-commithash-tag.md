---
title: "[Git] #2-4. 버전이 쌓여 사용자에게 선보이기까지: 커밋해시, 태그"
excerpt: "커밋해시와 태그 소스트리 실습"

categories:
  - Git
tags:
  - [Git, CommitHash, Tag]

permalink: /theory/git/commit-hash-and-tag

toc: true
toc_sticky: true

date: 2024-09-29
last_modified_at: 2024-09-29
---

## 커밋 해시

- 특정 커밋을 지칭하기 위해 각 버전에 부여된 고유한 정보로, 버전마다 다른 값을 가진다. 
- 문자열이 길기 때문에 커밋 해시의 앞 부분만 따서 짧은 커밋해시로 표기하기도 한다. 

![커밋해시](https://github.com/user-attachments/assets/5576972f-d45d-42bf-84a7-54256dd44e69)

---

## 태그

- 특정 커밋을 더 가독성 있게 지칭하기 위해 커밋에 붙이는 꼬리표이다. 
- 커밋 해시가 있는데 태그가 필요한 이유는 무엇일까? 여러 개의 커밋 중에서 이전 커밋보다 더 유의미하고, 분기점이 되는 커밋에 사용하기 때문에 필요하다. 

Sourcetree에서 태그를 붙여보자. 태그를 붙일 커밋에 우클릭을 한 후 태그를 붙인다. 삭제하는 방법도 우클릭 후 태그를 삭제하면 된다.

![image](https://github.com/user-attachments/assets/604b4029-9168-40be-b9fa-b859e87fdfff)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*