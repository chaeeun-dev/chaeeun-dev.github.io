---
title: "[Github Blog] Mimimal Mistake 블로그 테마, 색상 변경"
excerpt: "블로그 테마와 색상 변경하기"

categories:
  - Github Blog
tags:
  - [Github Blog, 테마, 색상]

permalink: /projects/github-blog/theme-color-custom

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---

## 기존 블로그 모습

블로그 색을 내가 좋아하는 색으로 바꾸고 싶어서 custom 방법을 알아봤다. 일단 기존 블로그는 이렇게 생겼다.

![image](https://github.com/user-attachments/assets/01ceb3b0-77c5-4cb7-9722-fa90c88c8ac7)

---

## minimal-mistakes - skins - custom2

(기존 custom1이 있어서) custom2.scss 파일을 새로 만들었다. 기존 air 테마(배경색이 너무... 구림...)에서 배경 색, 텍스트 색만 변경하면 딱 좋을 것 같아서 _air.scss 파일 내용을 그대로 복사했다. 

```markdown
$background-color           : #ffffff !default;
$text-color                 : #000 !default;
```

배경색과 텍스트색 둘만 변경했다. 어떤 부분의 색상을 고칠 것인지 알기 위해 참고한 블로그 [Mimimal Mistakes 블로그 스킨, 테마 변경](https://datanovice.tistory.com/entry/Minimal-Mistakes-Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EC%8A%A4%ED%82%A8-%ED%85%8C%EB%A7%88-%EB%B3%80%EA%B2%BD)

---

## _config.yml

```markdown
minimal_mistakes_skin    : "custom2"
```

---

## 변경된 블로그 모습

새로 만든 custom2를 skin에 넣어주면 이렇게 변경된다! 색상 변경은 좀 오래 기다려야 되는 것 같다. 그래도 5분 안에는 되니 다시 할 땐 조급해하지 말아야겠다.

![image](https://github.com/user-attachments/assets/92b4d9e1-2cc6-4799-b345-0e68722687ab)

