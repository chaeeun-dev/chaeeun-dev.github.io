---
title: "[GIT & GITHUB] 모두의 깃 & 깃허브 - 섹션3 버전 가지고 놀기"
date: 2024-09-30 12:50:00 +09:00 
categories: GIT
author_profile: false
toc : true
toc_sticky: true
---

## [Windows 실습] 버전 비교하기(24.09.30)

직전 버전과 비교해보자. 추가 됐으면 +표시와 초록색이, 삭제 됐으면 -표시와 빨간색이 표시된다.

![image](https://github.com/user-attachments/assets/f0828565-e64a-4b21-ae99-732db2bc903a) | ![image](https://github.com/user-attachments/assets/fe4c43ca-f460-41ce-b400-6ab5ba553466)


버전 별로 비교하고 싶을 때(예를 들어 두 번째 커밋과 다섯 번째 커밋의 차이를 비교)를 알아보자.

일단 두 번째 커밋 당시의 a.txt 파일을 확인할 수 있다. 두 번째 커밋을 클릭하고 a.txt를 우클릭 후 **선택한 버전 열기**를 클릭한다. 그러면 그 당시의 파일 내용을 확인할 수 있다. 

![image](https://github.com/user-attachments/assets/3bb98d2f-779a-4249-a1a1-0d962966289f)


그러면 두 번째 버전에서 다섯 번째 버전을 비교했을 때 달라진 점을 확인하는 방법을 알아보자. ctrl을 누르고 두 번째 버전과 다섯 번째 버전을 클릭한 후, 비교할 파일인 a.txt를 선택하면 우측 하단에 버전 차이를 확인할 수 있다.

![image](https://github.com/user-attachments/assets/195bb269-272d-4fe3-b756-0dfe57e559e5)

---

## [이론] 버전을 되돌리는 방법(24.10.01) 

만들어진 버전을 되돌리는 두 가지 방법
1. revert - 버전을 되돌린 새로운 버전 만들기
2. reset - 버전을 완전히 되돌리기

### revert
지금까지 만든 모든 버전이 유지된 채 새로운(이전 버전의 복제(?)) 버전을 만든다. **기존의 버전은 삭제되지 않는다.**

![image](https://github.com/user-attachments/assets/fca18fcb-d404-420f-8d89-a3e7977fa2e7)


### reset
되돌아갈 버전의 시점으로 완전하게 되돌아가는 방식이다. 즉, 되돌아갈 버전 이후의 버전은 삭제되는 방식이다.

하나의 버전이 만들어지는 과정 중
1. ~~작업 디렉터리에서 변경 사항 생성하기~~ → hard reset(작업 디렉터리까지 되돌리기)
2. ~~스테이지로 추가하기~~ → mixed reset(스테이지까지 되돌리기) 
3. ~~저장소로 커밋하기~~ → soft reset(커밋만 되돌리기)

---

## [Windows 실습] revert와 reset 실습(24.10.02)

### revert
마지막 커밋(7 번째)에서 다섯 번째 커밋으로 되돌아가고 싶을 때, 다섯 번째 커밋 우클릭 → **커밋 되돌리기**를 누른다. 

![image](https://github.com/user-attachments/assets/f9a07564-3541-4c67-bac1-c41946172a7c)

&nbsp;

'정말 커밋을 되돌리시겠습니까?'라는 창에 '예'를 선택한다.

![image](https://github.com/user-attachments/assets/68917d7a-e8f3-4700-83f3-6f9c978c354d)

&nbsp;

그럼 다섯 번째와 동일한 내용의 새로운 버전이 생성된다.


### reset

다섯 번째 커밋으로 reset하고 싶을 때, 다섯 번째 커밋 우클릭 → **이 커밋까지 현재 브랜치 초기화**를 누른다.

![image](https://github.com/user-attachments/assets/68ff094e-edc1-4b42-a886-4f377b33c650)

&nbsp;

soft reset, mixed reset, hard reset 중 어떤 리셋을 쓸건지 선택하고 확인을 누르면 된다. 나는 soft reset을 선택했기 때문에 다섯 번째 변경 사항을 스테이지로 추가한 과정까지 남아있다.

![image](https://github.com/user-attachments/assets/253748d7-36cd-4cdd-aac2-ab50ccd979b3)

&nbsp;

cf) Revert와 Reset을 제대로 할 수 없으면 "이 작업이 잘못되면 다 끝장이야.."라는 생각과 함께 작업에 소극적이게 될 수 있다. Revert와 Reset을 제대로 할 수 있으면 파일 날라가도 다시 되돌릴 수 있으니 쓸 데 없는 걱정이 줄어들게 되고, 작업을 더 과감하게 할 수 있게 된다!

---

## [Windows 실습] 작업 임시 저장하기 Stash(24.10.02)

개발 과정에서 작업 내역이 별로 마음에 들지 않지만 버리기는 아까울 때가 있을 수 있다.

![image](https://github.com/user-attachments/assets/b1fdc817-fa13-4da6-92db-1b35fbd22ac9)

&nbsp;

또는 갑자기 다른 더 중요한 일을 처리해야 할 때가 있을 수 있다.

![image](https://github.com/user-attachments/assets/555444dd-0df1-48ff-b26f-bb0848ef9148)

&nbsp;

이런 상황에서는 지금까지의 변경 내역을 지우기보다 임시 저장하는 것이 좋다. <mark style='background-color: #dcffe4'> 스태시 </mark>를 하면 모든 변경 사항이 임시 저장되고, 작업 디렉터리는 변경 사힝이 생기기 전의 깨끗한 상태로 돌아간다. 스태시는 여러 번 할 수 있으며 언제든 다시 꺼내 작업 디렉터리에 다시 적용할 수 있다. 그럼 Sourcetree에서 스태시 실습 해보자. 

변경 사항이 만들어졌으면, 상단의 **스태시**를 누른다.

![image](https://github.com/user-attachments/assets/6473e519-3a41-40ac-b0ab-7a37388d8bbf)

&nbsp;

이 창이 뜨면 설명을 작성하고(꼭 안 해도 됨) 확인을 누른다.

![image](https://github.com/user-attachments/assets/3249a4ec-ac7a-46a9-b54a-1c1e7fa655ce)

&nbsp;

좌측에 스태시 항목을 보면 임시저장1이 생겼다. 

a.txt에 C D를 추가했었는데, 스태시를 한 후 변경 사항이 생기기 전 상태로 파일이 복원되었다는 걸 알 수 있다.

![image](https://github.com/user-attachments/assets/62704366-354a-4ae6-8f62-20456324b59d)

&nbsp;

또한 스태시를 통해 두 개, 세 개 이상으로 저장할 수 있다.

이제 임시 저장된 변경 사항을 작업 디렉터리에 적용해보자. 임시 저장1에 우클릭 후 스태시 적용을 누른다.

![image](https://github.com/user-attachments/assets/ac9a3a87-3ef4-4b58-b87a-269b7ec7a84d)

&nbsp;

파일 상태를 보면, 임시 저장1의 변경 사항이 적용되었음을 알 수 있다.

![image](https://github.com/user-attachments/assets/fe3da5eb-efee-4afc-877f-cc491aacf5b7)

&nbsp;

---

출처 [인프런 강민철님 모두의 깃&깃허브](https://www.inflearn.com/course/%EB%AA%A8%EB%91%90%EC%9D%98-%EA%B9%83-%EA%B9%83%ED%97%88%EB%B8%8C)