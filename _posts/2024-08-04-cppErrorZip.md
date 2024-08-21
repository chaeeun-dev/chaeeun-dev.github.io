---
title: "[ERROR] VisualStudio 오류, 삽질 모음"
date: 2024-08-04 05:20:00 +09:00 
categories: ERROR
author_profile: false
---

### 24.08.21

삽질 1) GET_SINGLE(TimeManager)→Init(); 에서 → 이 부분에 “식이 필요합니다” 오류 나서 봤더니 Defines.h에 정의한 매크로에 ;를 붙여서였다. ( #define GET_SINGLE(classname) classname::GetInstance()  ← 여기에 세미콜론 붙여서)

삽질 2) 매크로에 단순 오타 Normal을 Nornal로 씀.
