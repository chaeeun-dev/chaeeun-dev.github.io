---
title: "[Git] #2-3. 버전 관리 맛보기: 버전 만들기 "
excerpt: "버전 만들기 소스트리 실습"

categories:
  - Git
tags:
  - [Git, VersionManage]

permalink: /theory/git/version-manage-practice

toc: true
toc_sticky: true

date: 2024-09-28
last_modified_at: 2024-09-28
---

## 버전 만들기

(.git이 있는) 작업 디렉터리에 a.txt 파일을 생성한다(지금은 텍스트 파일이지만 프로젝트 소스 코드 파일이라고 생각하며).

![image](https://github.com/user-attachments/assets/81e573a8-f7fc-4f90-a097-1028d1f328b9)


Sourcetree 앱을 켜보면 **스테이지에 올라가지 않은 파일**에 a.txt 파일이 있다. **스테이지에 올라가지 않은 파일**은 작업 디렉터리 내 변경 사항이지만, 아직 버전이 될 후보로 선정되지 않은 파일이다. 

![image](https://github.com/user-attachments/assets/233920f9-7656-4155-b22a-15e774a0848d)


이제 스테이지에 add를 하기 위해 **모두 스테이지에 올리기** 버튼을 누른다.

![image](https://github.com/user-attachments/assets/a5b6ba68-29b7-4143-ada8-b3a71441092d)


커밋 메세지: 버전에 붙이는 메모, 수많은 버전들이 생길텐데 그 **버전을 설명하는 메시지**이다. 제목+본문(본문은 생략 가능, 첫 줄은 제목 쓰고 한 줄 띄고 본문)으로 이루어져 있고, 반드시 길게 작성할 필요는 없다.  

커밋 메세지를 작성해보자.

![image](https://github.com/user-attachments/assets/ec3d604a-45f8-47c9-b68c-9f0faa8e315c)


메시지를 작성한 후 커밋을 누르면 버전이 생성된다. 모든 변경 사항을 스테이지로 올렸고, 스테이지에 있는 후보들을 모두 커밋했기 때문에 스테이지는 비어있게 된다(파일 상태에 "커밋할 내용 없음"이라고 뜬다). 

![image](https://github.com/user-attachments/assets/363a2ea7-0f6e-4359-8365-1fe00ae2d0fc)


두 번째로 a.txt에 두 번째 줄을 추가 저장 후, 스테이지에 올리고 커밋해본다.

![image](https://github.com/user-attachments/assets/b10f4a67-6089-4d74-a821-4e9029295b08)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*