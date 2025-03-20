---
title: "[Git] #4-6. 충돌 해결하기"
excerpt: "충돌 해결하기 실습"

categories:
  - Git
tags:
  - [Git, Collision]

permalink: /theory/git/collision-resolve-practice

toc: true
toc_sticky: true

date: 2024-10-09
last_modified_at: 2024-10-09
---

## 충돌 해결하기

충돌이란? 병합하려는 두 브랜치가 서로 같은 내용을 다르게 수정한 상황을 말한다. 충돌이 발생하면 브랜치가 한 번에 병합되지 못한다. 여럿이 협업할 때 빈번하게 발생하는 상황이니 꼭 해결 방법을 알아야 한다. 

---

### 충돌 예시 

master 브랜치에서 foo 브랜치가 뻗어나왔다. foo 브랜치는 a.txt 파일의 첫 번째 줄을 C로 수정한 후 커밋했고, master 브랜치는 a.txt 파일의 첫 번째 줄을 B로 수정한 후 커밋했다.

![image](https://github.com/user-attachments/assets/d1089314-bb4f-42ea-a13c-763f5fd979c4)

merge를 했을 때 어떤 커밋이 반영될까? 정답은 <span style="color:red"> 깃도 모른다! </span> 깃은 어떤 브랜치의 내용을 반영해야 하는지 판단할 수 없다. 

충돌이 발생했을 때는 어떤 브랜치의 내용을 반영할지 직접 선택해야 한다. 

---

### 충돌이 발생했을 때 대처법

1. 충돌을 해결한다(어떤 브랜치의 내용을 반영할지 직접 선별한다).
2. 다시 커밋한다.

---

### 충돌 해결하기 예시

master 브랜치에서 a.txt 파일을 생성 후 첫 번째 줄을 A로 수정한 후 커밋한다. 이제 foo 브랜치를 만들고 체크아웃하고 a.txt 파일의 첫 번째 줄을 foo로 수정한 후 커밋한다.

![image](https://github.com/user-attachments/assets/3742f34f-c753-410d-aa66-def47004d306)

이제 master 브랜치로 체크아웃한 후, a.txt의 첫 번째 줄의 A를 master로 수정하고 커밋한다.

![image](https://github.com/user-attachments/assets/6f064fbd-6585-45bb-9873-09a63ff1629a)


여기서 master 브랜치와 foo 브랜치를 merge해보자. master 브랜치로 체크아웃한 뒤 foo 브랜치를 우클릭해서 '현재 브랜치로 foo 병합'을 누른다. 

![image](https://github.com/user-attachments/assets/03d998c2-b40e-474a-a9fd-e264714b783b)


이렇게 충돌이 발생한다.

![image](https://github.com/user-attachments/assets/03d998c2-b40e-474a-a9fd-e264714b783b)


그래프에는 '커밋하지 않은 변경사항'이 뜬다.

![image](https://github.com/user-attachments/assets/17ffc5b2-c76c-45f0-ba8a-0a83d4d6afa8)


파일 탐색기에서 a.txt를 열어보면 내용이 이상하게 바뀌어 있다.

![image](https://github.com/user-attachments/assets/f211d843-7832-4c71-8058-030b59fb0b60)

이 내용은 깃이 남겨 놓은 표시이다. <<<<<<<< ~ ====== 이 부분은 master와 foo를 구분하기 위한 일종의 영역 표시!

![image](https://github.com/user-attachments/assets/1875f04e-b672-484c-b593-8c20d69df35e)


a.txt 우클릭 후 충돌 해결에 ['내것'을 이용해 해결], ['저장소'것을 사용하여 해결]이 있다. 번역이 이상하게 된 것인데, '내것'은 master 브랜치, '저장소'는 foo 브랜치를 선택하는 것이다. 여기서는 master 브랜치를 선택해보자.

![image](https://github.com/user-attachments/assets/46ad5c3e-d8f0-4102-990b-14a34775ef86)


그럼 master 브랜치의 커밋으로 a.txt가 수정되었다.

![image](https://github.com/user-attachments/assets/a7a69abd-1cbb-4696-8b8b-0aee45282b0c)


근데 아직 merge가 안 되었으니, 파일 상태에서 자동으로 작성된 커밋 메세지로 커밋해준다.

![image](https://github.com/user-attachments/assets/bdd17668-65ab-41c6-b7ab-238834005adb) | ![image](https://github.com/user-attachments/assets/e1a9cc53-d134-4b60-be6f-2eac129b8fe0)


그럼 충돌이 해결되면서 병합까지 완료 되었다!

![image](https://github.com/user-attachments/assets/8f402cf4-d1cc-4715-b965-ffd6fb1b4fa0)

---

## 마치며

**충돌이 발생했다고 당황하지 말자. 그냥 해결하면 된다~!**

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*