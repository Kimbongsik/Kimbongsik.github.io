---
title: "[보안] Stack Frame이란"
last_modified_at: 2023-02-27T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - security

tags:
  - security
---

보안 공부를 하다보면 Stack Frame이라는 개념을 마주치게 된다.  
메모리의 스택 영역에는 지역 변수와 매개변수가 저장되는데, 그렇기 때문에 함수가 호출되면 스택에 함수의 매개변수와 return 주소값, 함수 내에 선언된 지역 변수가 저장된다. 이렇게 스택 영역에 저장되는 함수의 호출 정보를 스택 프레임이라고 한다.  
스택 프레임으로 인해 함수의 호출이 끝난 후 함수가 호출되기 이전 상태로 돌아갈 수 있다. 

<br/>

```c
#include <stdio.h>

int sum(int a, int b){
	return a+b;
}

int main(void){
	int c = sum(1, 2);
	return c;
}
```
*명령어:  
>gcc - S -fno-stack-protector -mpreferred-stack-boundary=4 -Z execstack -o sum.a sum.c

*gcc의 스택 취약점 방어 기능 해제

*64비트 환경에서 컴파일 하는 것을 명시(stack boundary = 4)

- -mpreferred-stack-boundary = N
    - main을 포함하여 이후에 불리는 모든 함수들의 스택 프레임에서 가장 낮은 주소가 2^N에 배수가 되도록 align 해줌

![1](https://user-images.githubusercontent.com/63995044/221567545-a72dc31f-6651-478e-b609-65eac64284a5.png)

스택의 맨 마지막: RET(Return Address)

스택의 가장 밑바닥: RBP

>main에서 sum 함수 호출 시

![2](https://user-images.githubusercontent.com/63995044/221567553-ff92f916-bc23-462b-a273-3d4e5f8b7438.png)

- 빨간색: sum() 호출 이후에 새로 쌓인 sum() 함수의 스택 프레임
- 파란색: 기존의 main 함수의 스택 프레임

main 함수가 끝나기 전에 sum 함수를 불러왔기 때문에 main 함수 위로 sum 함수가 쌓임

이후에 sum 함수가 할 일을 마치고 리턴하기 되면 위에서부터 pop 하여 스택에는 main 함수의 스택 프레임만 남게 됨

- 빨간 부분의 sum 함수 스택 프레임
    - RET: sum을 다 실행한 후 돌아가야 할 위치
    - x, y: sum 함수의 매개 변수(스택에 RET보다 먼저 쌓인다)
    - RBP: 스택 시작, Buffer: 쌓인다
- 파란 부분의 main 함수 스택 프레임
    - 변수 c에 sum 함수가 리턴되어 결과값인 3이 담김