---
title: "[Git] #4-5. 브랜치 합치기"
excerpt: "브랜치 합치기 실습"

categories:
  - Git
tags:
  - [Git, CombineBranch, Practice]

permalink: /theory/git/combine-branch-practice

toc: true
toc_sticky: true

date: 2024-10-08
last_modified_at: 2024-10-08
---

## 브랜치 합치기

![image](https://github.com/user-attachments/assets/86fd4460-9b50-4e38-b387-2517618cfb3a)

master 브랜치에서 뻗어나온 foo 브랜치는 한 개의 commit이 쌓였다. 그동안 master 브랜치에는 어떠한 커밋도 없었다. ➡️ foo 브랜치를 master 브랜치에 병합시키면, <span style="color:red"> fast-forward merge </span>가 된다.

---

### fast-forward merge 방법 예시

foo 브랜치를 master 브랜치에 병합해보자.

- master 브랜치를 더블 클릭해서 <span style="color:red"> 체크아웃 </span>한다.

![image](https://github.com/user-attachments/assets/b390bb31-094d-4099-abae-b0ed587ac50c)

- foo 브랜치에 우클릭 후 '현재 브랜치로 foo 병합'을 누른다.

![image](https://github.com/user-attachments/assets/64b9bab0-8d6c-465f-b9fe-0a90f0885e63)

&nbsp;

그럼 병합이 완료 됐다!

![image](https://github.com/user-attachments/assets/da56ab04-55ad-4e07-ba6e-e76e4218a7a3)

&nbsp;

파일 탐색기를 열어보면 foo 브랜치에서 작업했던 'foo_a.txt' 파일이 잘 보인다.

![image](https://github.com/user-attachments/assets/a5040483-9f15-4368-8fa9-70d62c8e3990)

&nbsp;

병합이 완료됐으니, foo 브랜치를 삭제해주자.

![image](https://github.com/user-attachments/assets/d763bbdd-16e5-4447-bf72-b7899a9e3de2)

---

### 일반적인 merge 방법 예시

이번에는 bar 브랜치를 생성하고 체크아웃 해본다.

![image](https://github.com/user-attachments/assets/c03e0786-8a57-475b-9583-c71b5d529ea2)

bar 브랜치에 'bar_a.txt' 파일을 생성하고 커밋 메세지를 'bar 4'로 커밋한다.

이번에도 bar 브랜치에 'bar_b.txt' 파일을 생성하고 커밋 메세지를 'bar 5'로 커밋한다.

![image](https://github.com/user-attachments/assets/7ab73aa1-ab3e-4aae-9e80-9e270372b122)

master 브랜치에는 커밋이 3개, bar 브랜치에는 커밋이 5개 쌓여있다.

&nbsp;

이제 master 브랜치에도 작업을 쌓도록 하자.

master 브랜치로 체크아웃 한 후, 'c.txt' 파일을 추가하고 커밋 메세지를 'master 4'로 커밋한다.

![image](https://github.com/user-attachments/assets/e9fec8dc-fc37-4604-9159-0b6c4cec0e4a)

그래프를 해석해보자.

점 하나가 커밋인데, master 브랜치에서 커밋이 3개 만들어진 후 bar 브랜치가 뻗어 나왔고 커밋이 2개 쌓였다. 그 사이 master 브랜치는 세 번째 커밋을 기반으로 한 네 번째 커밋을 만들었다.

이제 bar 브랜치를 master 브랜치로 merge해보자(이 merge는 fast-forward merge가 아님!).

- master 브랜치로 체크아웃 한다.

- bar 브랜치에 우클릭 후 '현재 브랜치로 bar 병합'을 누른다.

![image](https://github.com/user-attachments/assets/88e491b2-ca0f-4464-9833-67ed2d35efbe)


그럼 master 브랜치로 병합이 완료됐고, 이런 그래프가 나타난다!

![image](https://github.com/user-attachments/assets/4b6a534c-7fae-48ad-a68c-cfda89cf876b)


파일 탐색기를 열어보면, bar 브랜치에서 생성한 'bar_a.txt'와 'bar_b.txt' 파일이 잘 보인다. 

![image](https://github.com/user-attachments/assets/032e3d28-ecc0-4169-92e2-b4d884a61888)

병합이 완료됐으니, 필요없는 bar 브랜치는 삭제해도 된다.


브랜치 합치기 실습 끝!

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*