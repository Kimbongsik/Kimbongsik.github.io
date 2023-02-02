---
title: "[C언어] 2차원 배열 동적 할당"
last_modified_at: 2023-02-02T16:20:02-05:00
categories:
  - C_C++

tags:
  - C/C++
---


정적으로 배열을 생성할 때는

```c
int arr[5][10];
```

이런 식으로, 정수형 변수가 저장되는 가로 5, 세로 10 크기의 2차원 배열을 생성한다.

동적으로 배열을 생성할 때는 다음과 같이 생성한다.

```c
int height = 5;
int widgth = 10;

int **arr;

arr = (int**) malloc (sizeof(int *) * height);
for(int i =0; i<height; i++){
	arr[i] = (int*) malloc(sizeof(int) * weight);
}
```

동적으로 할당받은 메모리도 배열처럼 접근이 가능하다.

*일차원 배열 동적 할당

```c
int height = 5;
int *arr;
arr = (int*)malloc(sizeof(int *) * height);
```
