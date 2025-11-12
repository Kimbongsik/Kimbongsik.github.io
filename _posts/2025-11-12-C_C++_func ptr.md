---
title: "[C/C++] 함수 포인터 정의 및 응용 방법"
last_modified_at: 2025-11-12T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - C_C++

tags:
  - C_C++
---
## 1. 함수 포인터란?

- 함수를 가리키는 변수로, 함수의 주소를 저장하고 있음
- 일반적으로 함수를 호출할 때, 함수가 매핑된 메모리 주소로 점프하게 되어 있음
- 함수 포인터는 이러한 함수의 메모리 주소를 저장하는 변수로, 아래와 같이 사용됨

```c
int(*funcPtr)(); // int를 반환하는 함수에 대한 포인터
```

- int 포인터를 반환하는 함수인 “int* funcPtr()” 와 구별하기 위해, 우선순위를 위해 저렇게 표기하는 것

## 2. 사용 방법

### 할당

```c
int foo(){ return 10; }

int goo(){ return 20; }

int main(){
	int(*funcPtr)() = foo; //foo 주소를 가리키는 함수 포인터
	funcPtr = goo; //goo 주소를 가리키는 함수 포인터
	
	return 0;
}
```

### 호출

```c
int foo(int x){ return x; }

int main(){

	int (*funcPtr)(int) = foo;
	(*funcPtr)(5); //foo(5); 와 동일한 기능 수행
	funcPtr(5); // 암시적 역참조를 통한 호출. 위와 기능 동일
	
	return 0;
}
```

## 3. 응용 방법

### 함수 인자로 전달(실행 주소 변경)

- 어떤 함수의 인자로써 함수를 전달하여 사용할 수 있으며, 이때 인수로 사용되는 함수를 콜백 함수라고 함
- 기능 별로 함수를 구현해두고, 해당 함수를 함수 인자로 전달하면 인자 변경을 통해 기능 스위칭을 간편하게 수행할 수 있음

```c
void SelectSort(int* array, int size, bool(*comparisonFunc)(int, int)){
	for (int i = 0; i < size; ++i){
		int min = i;
		for(int j = i + 1; j <size; ++j){
			if(comparisonFunc(array[min], array[j]))
				min = j;
			}
			
			std::swap(array[min], array[j]);
	}
}

bool ascending(int x, int y){ return x > y }

bool descending(int x, int y){ return x < y }

int main(){
	int array[9] = {3,5,7,6,2,8,9,1,4};
	
	// ascending sort
	SelectSort(array, 9, ascending);
	//descending sort
	SelectSort(array, 9, descending);
	
	return 0;
}
	
```

### 실행 주소 변경

- 함수 포인터를 통해 소프트웨어 실행 중 원하는 지점으로 branch 할 수 있음
- 임베디드 시스템 개발 시 프로그램 실행 흐름을 제어하기 위해 종종 사용됨

```c
void (*func)(void); // 함수 포인터 선언
func = (void(*)())0x0800; // 주소 포인팅 (함수 포인터 캐스팅)
(*func)(); // 호출
```

또는,

```c
(*(void(*)())0x0800)(); //호출
```