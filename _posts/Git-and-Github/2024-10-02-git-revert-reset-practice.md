---
title: "[Git] #3-3. revert와 reset 실습"
excerpt: "revert와 reset 소스트리 실습"

categories:
  - Git
tags:
  - [Git, Revert, Reset, SourceTree]

permalink: /theory/git/revert-reset-practice

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---

## revert

마지막 커밋(7 번째)에서 다섯 번째 커밋으로 되돌아가고 싶을 때, 다섯 번째 커밋 우클릭 → **커밋 되돌리기**를 누른다. 

![image](https://github.com/user-attachments/assets/f9a07564-3541-4c67-bac1-c41946172a7c)

'정말 커밋을 되돌리시겠습니까?'라는 창에 '예'를 선택한다.

![image](https://github.com/user-attachments/assets/68917d7a-e8f3-4700-83f3-6f9c978c354d)


그럼 다섯 번째와 동일한 내용의 새로운 버전이 생성된다.

---

## reset

다섯 번째 커밋으로 reset하고 싶을 때, 다섯 번째 커밋 우클릭 → **이 커밋까지 현재 브랜치 초기화**를 누른다.

![image](https://github.com/user-attachments/assets/68ff094e-edc1-4b42-a886-4f377b33c650)

soft reset, mixed reset, hard reset 중 어떤 리셋을 쓸건지 선택하고 확인을 누르면 된다. 나는 soft reset을 선택했기 때문에 다섯 번째 변경 사항을 스테이지로 추가한 과정까지 남아있다.

![image](https://github.com/user-attachments/assets/253748d7-36cd-4cdd-aac2-ab50ccd979b3)

---

## 마치며

Revert와 Reset을 제대로 할 수 없으면 "이 작업이 잘못되면 다 끝장이야.."라는 생각과 함께 작업에 소극적이게 될 수 있다. Revert와 Reset을 제대로 할 수 있으면 파일 날라가도 다시 되돌릴 수 있으니 쓸 데 없는 걱정이 줄어들게 되고, 작업을 더 과감하게 할 수 있게 된다!

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*