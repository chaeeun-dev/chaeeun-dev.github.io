---
title: "[Git] #3-4. 작업 임시 저장하기"
excerpt: "스태시 실습"

categories:
  - Git
tags:
  - [Git, Stash]

permalink: /theory/git/stash-practice

toc: true
toc_sticky: true

date: 2024-10-03
last_modified_at: 2024-10-03
---

## 스태시가 필요한 이유

- 개발 과정에서 작업 내역이 별로 마음에 들지 않지만 버리기는 아까울 때가 있을 수 있다.

![image](https://github.com/user-attachments/assets/b1fdc817-fa13-4da6-92db-1b35fbd22ac9)

- 또는 갑자기 다른 더 중요한 일을 처리해야 할 때가 있을 수 있다.

![image](https://github.com/user-attachments/assets/555444dd-0df1-48ff-b26f-bb0848ef9148)

이런 상황에서는 지금까지의 변경 내역을 지우기보다 임시 저장하는 것이 좋다. <mark style='background-color: #dcffe4'> 스태시 </mark>를 하면 모든 변경 사항이 임시 저장되고, 작업 디렉터리는 변경 사힝이 생기기 전의 깨끗한 상태로 돌아간다. 스태시는 여러 번 할 수 있으며 언제든 다시 꺼내 작업 디렉터리에 다시 적용할 수 있다. 그럼 Sourcetree에서 스태시 실습 해보자. 

---

## 작업 임시 저장하기 실습

변경 사항이 만들어졌으면, 상단의 **스태시**를 누른다.

![image](https://github.com/user-attachments/assets/6473e519-3a41-40ac-b0ab-7a37388d8bbf)


이 창이 뜨면 설명을 작성하고(꼭 안 해도 됨) 확인을 누른다.

![image](https://github.com/user-attachments/assets/3249a4ec-ac7a-46a9-b54a-1c1e7fa655ce)


좌측에 스태시 항목을 보면 임시저장1이 생겼다. 

a.txt에 C D를 추가했었는데, 스태시를 한 후 변경 사항이 생기기 전 상태로 파일이 복원되었다는 걸 알 수 있다.

![image](https://github.com/user-attachments/assets/62704366-354a-4ae6-8f62-20456324b59d)


또한 스태시를 통해 두 개, 세 개 이상으로 저장할 수 있다.

이제 임시 저장된 변경 사항을 작업 디렉터리에 적용해보자. 임시 저장1에 우클릭 후 스태시 적용을 누른다.

![image](https://github.com/user-attachments/assets/ac9a3a87-3ef4-4b58-b87a-269b7ec7a84d)

파일 상태를 보면, 임시 저장1의 변경 사항이 적용되었음을 알 수 있다.

![image](https://github.com/user-attachments/assets/fe3da5eb-efee-4afc-877f-cc491aacf5b7)

--- 

*출처*
*[강민철님 모두의 깃 & 깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)*