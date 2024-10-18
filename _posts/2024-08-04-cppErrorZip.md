---
title: "[ERROR] VisualStudio 오류, 삽질 모음"
date: 2024-08-04 05:20:00 +09:00 
categories: ERROR
author_profile: false
toc : true
toc_sticky: true
---

## 24.08.21

### 매크로에 ; 붙여서 

GET_SINGLE(TimeManager)→Init(); 에서 → 이 부분에 “식이 필요합니다” 오류 나서 봤더니 Defines.h에 정의한 매크로에 ;를 붙여서였다. ( #define GET_SINGLE(classname) classname::GetInstance()  ← 여기에 세미콜론 붙여서)

### 오타

매크로에 단순 오타 Normal을 Nornal로 씀.



## 24.08.28

### return 값 제대로 확인하지 않아서 

스프라이트 강의() 듣던 중 리소스를 화면에 그리는 코드를 작성헀는데, 제대로된 경로임을 확인하고도 화면에 그림이 로드되지 않는 상황이 발생했다.

![image](https://github.com/user-attachments/assets/bf060abf-f846-4115-b54b-42691cca675d)

원인) 오류의 원인은 아주 사소한 문제였다. 경로 문제는 역시 아니었고, Texture cpp 파일에서 HDC Texture::GetDC() 함수에 return HDC();를 return _hdc;로 바꾸지 않았기 때문! LoadBmp() 함수에서 _hdc를 수정했는데, 그 수정된 _hdc를 GetDC 함수에서 내보내야 하는데 이상한 값을 보내고 있었다.

결론) ctrl + . + enter로 클래스에서 함수를 생성하면 기본적으로 return 값을 넣어주는 경우가 있는데, 또 삽질을 경험할 수 있으니.. 코드가 들어가 있으면 바로 지워버리자

![image](https://github.com/user-attachments/assets/ddb99b76-fa0f-4de1-8086-23c27e85f78b)


### class에서 public 안 붙여서 

![image](https://github.com/user-attachments/assets/401fc8b9-913f-4fff-8aca-978e13052a50)

class에서 생성자에 public을 안 붙여서!!!





## 24.09.08

### 미해결 오류(프로젝트를 새로 파서 묻혀진 오류...)

![image](https://github.com/user-attachments/assets/5a6ab72d-1c8e-46f3-9494-a213bf69e225)

문제) 타일맵 강의 들으면서 devScene.cpp에 타일 생성하고 맵에 그려주려고 하는데 자꾸 this가 nullptr이라는 오류가 발생한다!! 아직 해결 못함.. 이틀째..

해결 과정)
![image](https://github.com/user-attachments/assets/b038b923-3906-4405-ada2-460bbdde0a74)
호출스택에서 TilemapActor.cpp 77번째 줄(TransparentBlt 함수)에서 Sprite::GetTransparent()를 호출했음을 확인


## 24.09.30

### 미리 컴파일된 헤더 pch

C1083 미리 컴파일된 헤더 파일을 열 수 없습니다. 

![image](https://github.com/user-attachments/assets/e0ca05ea-48bc-498b-b5a7-605d1edfbaf5)


해결) 

1. 프로젝트 속성 - C/C++ - 미리 컴파일된 헤더 - 미리 컴파일 된 헤더 **사용(/Yu)** 선택, 미리 컴파일된 헤더 파일에 **pch.h** 입력
2. pch.cpp 파일 속성 - 미리 컴파일된 헤더 **만들기(/Yc)**

참고 블로그 [[C/C++] 미리 컴파일된 헤더](https://noirstar.tistory.com/12)


## 24.10.05

### thread에서 join() 함수 안 써서

오류) 쓰레드 관련 코드를 실행했는데, "abort() has been called" 오류가 발생했다.

```cpp
	std::thread t1(Add);
	std::thread t2(Sub);
```

![image](https://github.com/user-attachments/assets/ff74626a-0cdb-4dfb-9654-c9c666e7f469)

'다시 시도(R)'를 누르면 오류 위치를 알 수 있다(VS가 자동으로 중단점을 트리거 해줌).

![image](https://github.com/user-attachments/assets/1d64087f-7e23-4eef-af3b-59742c4920c0)

해결) 쓰레드 실행하고 join() 함수 안 넣어서 오류 발생했던 것이다. 확인 꼼꼼히 하자!! 그래도 문제가 발생했을 때 '다시 시도'를 누르면 중단점으로 간다는 걸 처음 알게됐다.

```cpp
	std::thread t1(Add);
	std::thread t2(Sub);

	t1.join();
	t2.join();
```

## 24.10.13

### VS 업데이트 후 미리 컴파일된 헤더 오류

C2857	/Yc<다른 옵션>pch.h 명령줄 옵션과 함께 지정한 '#include' 문이 소스 파일에 없습니다.

비주얼 스튜디오 업데이트 했는데 잘 돌아가던 코드에 오류가 발생했다. 또 pch.h 오류다 ㅎㅎㅎ

![image](https://github.com/user-attachments/assets/d01b77fb-8c9f-4c57-9a3c-e15f0886cba1)

해결 과정)

- 구글링을 하다가 비주얼 스튜디오를 다시 업데이트 해보라는 글이 있어서 다시 해보려고 한다. [vs 업데이트 한 후, 프로젝트 로딩이 안 될 경우 : 다시 업데이트 필요](https://blog.naver.com/m_jackson_ko/222425792451) ➡️ 해결 안 됨.

- 이미 깃허브에 레파지토리 올려놨으니 삭제하고 Clone(Open with Visual Studio)했더니 잘 실행이 된다. 정확한 원인은 모르겠는데 아마 업데이트 때문에 꼬인 것 같다. 잘 되던 게 안 되면 삭제했다가 다시 클론해보자! ➡️ 해결 완료!

## 24.10.18

### VS code 커밋 conflict
