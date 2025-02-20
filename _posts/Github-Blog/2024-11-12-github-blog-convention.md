---
title: "[Github Blog] 블로그 컨벤션 및 구분선 없애기"
excerpt: "블로그 컨벤션과 구분선 없애는 방법"

categories:
  - Github Blog
tags:
  - [Github Blog, 컨벤션]

permalink: /projects/github-blog/blog-convention

toc: true
toc_sticky: true

date: 2024-11-12
last_modified_at: 2024-11-12
---

## 컨벤션 정리 이유
글 구조에 대한 컨벤션을 안 정해 놓으니 글이 너무 지저분해 보이고, 글 쓸 때마다 개행을 어떻게 해야할지 등에 대한 고민이 자꾸 들었다. 글 구성, 목차, 개행 등을 어떻게 작성할지 정해서 글을 깔끔하게 작성하고 싶었다.

---

## 컨벤션
- 글 작성

![image](https://github.com/user-attachments/assets/cdf2f200-7656-496f-838c-5a66ba34d4ad)

- 이모티콘 
목차에 이모티콘 사용 X(TIL 제외), 글에 강조하고 싶은 내용 있으면 번거롭더라도 글자 색 변경하기.

(저번에 목차에 사용해봤는데 하나도 안 깔끔해 보여서 그냥 사용하지 않기로 했다. 그리고 글 강조할 때 이모티콘 쓰니까 더 안 깔 끔해 보이니 글자 색을 변경하자.)

- 사진 삽입
사진 삽입 후 개행 들이기.

- ```&nbsp;```
구분이 필요할 때만 사용하기.

---

## 자동 구분선 없애기
글이 지저분해 보였던 이유를 찾았다. ```## 목차``` 를 쓰면 아래 구분선이 그어지는데 minimal-mistakes 테마에서 그렇게 정의하고 있던 것이었다. 

```
.page__content {
  h2 {
    padding-bottom: 0.5em;
    border-bottom: 1px solid $border-color;
  }
```

_page.scss에서 원래 이렇게 정의 되어 있다.

&nbsp;

```
.page__content {
  h2 {
    padding-bottom: 0.5em;
    border-bottom: none; // 구분선 제거
  }
}

```

구분선을 제거하기 위해 ```border-bottom``` 을 ```none```으로 설정했다.

이제 h2를 작성해도 구분선이 보이지 않는다!!! 

***
