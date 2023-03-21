---
title: "[C/C++] ERROR ‘for’ loop initial declarations are only allowed in C99 or C11 mode 원인 및 해결"
last_modified_at: 2023-03-21T16:20:02-05:00
categories:
  - C_C++

tags:
  - C/C++
---

## 상황

c로 짜여진 파일을 gcc로 컴파일 할 때 해당 오류가 발생하는 경우가 있다.

오류 구문: for(int i=0; i<pos; i++)

## 원인

for문의 초기식에 정의를 하는 방식은 “loop initial declarartion”이라고 하며 이는 C99 방식에서 사용하는 것이다. 따라서 for문의 초기식에서 변수를 정의하기 위해선 C99을 지원하는 환경이어야 한다. 그러나 GCC는 C99이 기본적으로 설정되어 제공되지 않기 때문에, 당연히 에러가 발생하게 되는 것이다. 이 에러는 컴파일러 설정을 C99이나 C11 표준으로 바꿔줄 것을 안내하고 있다. 

## 해결

변수 i의 정의를 for문 바깥에 해주면 된다.
