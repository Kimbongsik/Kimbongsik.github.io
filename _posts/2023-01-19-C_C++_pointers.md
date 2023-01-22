---
title: "[C/C++] 포인터 사용 예제"
last_modified_at: 2023-01-17T16:20:02-05:00
categories:
  - C_C++

tags:
  - C/C++
---

<aside>
💡 해당 포스트는 아래 블로그의 내용을 통해 공부한 내용을 정리한 포스트입니다.

</aside>

>[https://dev-ku.tistory.com/101](https://dev-ku.tistory.com/101)

>[https://reakwon.tistory.com/33](https://reakwon.tistory.com/33)

# 함수 포인터

> **개념**
> 
- 함수의 이름은 곧 메모리 주소(배열과 동일)
- 함수 호출 시 함수가 저장된 주소로 이동
- 함수의 주소 또한 포인터에 저장 가능하며, 따라서 포인터를 통해 함수를 호출하는 것이 가능

### 함수 포인터 형태

```c
int (*fp)(int, int); //포인터 선언
```

- 함수 포인터의 변수명은 괄호로 묶어야 함
- 괄호가 없으면 주소를 반환하는 함수의 선언

### 사용 예제 1

```c
#include <stdio.h>

int sum(int, int);

int main(void){
	int (*fp)(int, int);
	fp = sum; //포인터 초기화
	int result = fp(30,40);
	printf("result: %d\n", result);
	return 0;
}

int sum(int a, int b){
	return (a+b);
}
```

- 함수 포인터를 매개변수로 사용하는 것 또한 가능하다.

### 사용 예제 2

```c
#include <stdio.h>

void func(int (*fp)(int, int)){ //함수 포인터를 매개변수로 받는 함수
	int a, b;
	scanf("%d %d", &a, &b);
	printf("result: %d\n", fp(a,b));
}

int sum(int a, int b){return(a+b);}
int mul(int a, int b){return(a*b);}

int main(void){
	func(sum);
	func(mul);
}
```

# void 포인터

> **개념**
> 
- 어떤 형태의 주소든 저장할 수 있음
- 자료형을 알 수 없으므로, 간접 참조 연산이나 정수를 더하는 포인터 연산이 불가능
- 형 변환을 통해 포인터 변수의 자료형을 일시적으로 바꾸어 간접 참조 연산이나 주소값 연산 가능

### 사용 예제 3

```cpp
void *vfp;
int a = 10;
vfp = & a;
printf("%d\n", *(int *)vfp); //int 형변환. == 10
```

### 사용 예제 4

```cpp
#incdlue <stdio.h>

int main(){
	int arr[3] = {1,2,3};
	void *vfp = arr;

	for(int i=0; i<3; i++){
		printf("%d\n", *((int *)vfp) + i)); // 1 2 3
	}
	return 0;
}
```

# 포인터 배열 & 배열 포인터

> **포인터 배열 개념**
> 

```cpp
int *pArr[3] = {&a, &b, &c};
```

- pArr[0] = a의 주소 / *pArr[0] = a
- pArr[1] = b의 주소 / *pArr[1] = b
- pArr[2] = c의 주소 / *pArr[2] = c

- 즉, 배열의 요소가 포인터로 이루어져 있음

> **배열 포인터 개념**
> 

```cpp
int arr[3] = {1,2,3};
int (*pArr)[3] = arr; //int형 요소를 3개 가지고 있는 배열을 가리키는 포인터
```

- 특정 배열을 가리킬 수 있는 한 개의 포인터