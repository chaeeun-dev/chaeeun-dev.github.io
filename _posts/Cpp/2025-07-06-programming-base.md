---
title: "[C++] 프로그래밍 기초"
excerpt: ""

categories:
  - C++
tags:
  - [C++]

permalink: /C++/cpp-base/

toc: true
toc_sticky: true

date: 2025-07-06
last_modified_at: 2025-07-06
---

## 프로그램의 기본 요소

- 식별자(identifier)
    - 어떤 대상을 유일하게 구별할 수 있는 이름
    - 프로그램에서 사용되는 변수나 함수, 구조체나 클래스는 모두 고유의 식별자를 가져야 한다.
    - 식별자의 시작은 반드시 문자나 '_'

&nbsp;

- 키워드(keyword)
    - 특별한 의미가 주어진 식별자
    - 변수나 함수 등의 식별자로 사용할 수 없다.
    - 예. class, bool, case, if/else, inline, goto, auto 등
    - 키워드를 식별자로 사용하지 않는 것이 안전하다.

---

## 변수, 상수, 자료형

- 변수(variable)
    - 어떤 값을 저장하기 위한 메모리 공간
    - 변수는 이름을 가지며 특정한 자료형의 값을 저장할 수 있는 공간을 갖고, 안에는 어떤 값이 들어있다. 
    - 변수의 값은 언제든지 변경할 수 있고, 사용 전에 초기화해야 한다.
    - 주소 추출 연산자 &를 적용해 그 변수의 주소를 알 수 있다.

&nbsp;

- 상수(constant)
    - 변경될 수 없는 자료
    - 상수는 숫자보다 이름을 붙여 사용하는 것이 좋은데, 이를 기호상수(symbolic constant) 또는 리터럴(literal)이라고 한다.
    - 두 가지 방법으로 상수를 표현할 수 있다.
        ```cpp
        #define PI 3.141592 // 전처리기 문장을 사용한 상수 PI 선언
        const double PI = 3.141592  // const 키워드를 사용한 상수 PI 선언

        double area = PI*radius*radius  // 리터럴 PI 사용 예시
        ```
    - 상수는 값을 바꿀 수 없으며 메모리 공간을 차지하지 않으므로 주소가 없다. → 주소 추출 연산자 적용 불가능

&nbsp;

- 자료형(data type)
    - 기본 자료형(basic type)과 유도 자료형(derived type)으로 나뉜다.
        - 정수형 - char(1), short(2), int(4), long(4)
        - 실수형 - float(4), double(8)
        - 부울형 - bool(1)
    - 모든 정수 자료형 앞에 unsigned를 붙이면, 동일한 공간으로 0을 포함한 양수만 표현한다. short는 -32768~32767, unsigned short는 0~65535의 범위를 표현한다.

---

*출처* 
*[게임으로 배우는 C++](https://www.booksr.co.kr/product/%EA%B2%8C%EC%9E%84%EC%9C%BC%EB%A1%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-c/)*