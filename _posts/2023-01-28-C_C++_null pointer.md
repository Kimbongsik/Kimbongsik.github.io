---
title: "[C/C++] 널 포인터(Null pointer)"
last_modified_at: 2023-01-28T16:00:00-05:00
categories:
  - C_C++

tags:
  - C/C++
---


C++에서 포인터로 null 값을 지정하려면 리터럴 0으로 초기화하거나 할당하면 된다.

```cpp
float* ptr{0}; // ptr is a null pointer

float*ptr2; //ptr2 is uninitalized
ptr2 = 0; //ptr2 is now a null pointer
```

포인터가 null이면 bool값 false로, 아니면 true로 반환되기 때문에 이를 이용하여 포인터가 null인지 아닌지를 확인한다.

```cpp
doulbe *ptr{0};

if (ptr)
	cout << "ptr is pointing to a double value" << endl;
else
	cout << "ptr is a null pointer" << endl;
```

그럼 Dereferencing을 할 때는 어떻게 해야 할까? null 포인터를 역참조할 경우 응용 프로그램이 중단된다. 포인터 역참조는 포인터가 가리키는 주소로 이동하여 그 값에 접근한다는 것을 의미하는데, null 포인터의 경우 가리키는 주소가 없으므로 난감해진다. 


# null macro (in C)

C언어에는 값 0으로 defined 된 NULL이라는 특수 전처리기 매크로가 있다. C++에서 사용이 가능하긴 하지만 전처리기 매크로이고 어쨌거나 C의 기능이므로.. 권장되지는 않다고 한다.

```c
double* ptr{NULL}; // assing address 0 to ptr
```


# nullptr (in C++11)

값 0은 포인터 유형이 아니기 때문에 포인터가 null 포인터임을 나타내기 위해 포인터에 0을 할당하는 것은 일관성이 없다. 드문 경우이긴 해도 리터럴 0을 사용할 경우 컴파일러에서 null 포인터를 의미하는지 정수 0을 의미하는지 알 수 없어 문제가 발생할 수 있다고 한다.

```cpp
doSomething(0);
```

그래서~

C++11에서는 nullptr이라는 새로운 키워드를 도입했다.

이런 식으로 쓰면 된다.

```cpp
int *ptr {nullptr}; 
```
