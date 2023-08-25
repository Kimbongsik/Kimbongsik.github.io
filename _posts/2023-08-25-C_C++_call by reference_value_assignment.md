---
title: "[C/C++, Python] Call by reference, Call by value and Call by assignment(파이썬 여러 파일에서의 전역 변수 접근 및 수정)"
last_modified_at: 2023-08-25T16:20:02-05:00
categories:
  - C_C++

tags:
  - C/C++
  - Python
---

<br>

오늘은 좀 생뚱맞게 Python을 다뤄보려고 한다.

왜냐면 C와 C++이 더 익숙한 나에게 파이썬의 이것 때문에 오늘 삽질을 좀 했기 때문이다. 하..

오늘의 주제는 ‘여러 파일에서의 전역변수 수정’이다.

아무튼 그럼 바로 시작~!

<br>

# Call by value

- 값을 복사하여 전달

예를 들어 함수의 인자(argument)를 받을 때, 변수에 담긴 값 자체를 스택에 복사하여 넘겨준다. 

변수 a가 선언되어 있고, 어떤 함수 func의 인자로 a를 넘겨줄 때, func(a)에서 전달받은 a는 원래 변수인 a 그 자체가 아니라 a의 복사값이라는 의미이다. 따라서 함수 내에서 a를 바꾸었다고 해도, 실제 a 값은 변하지 않는다.

함수를 써서 전역 변수를 바꾸기 위해서는 리턴 값을 전역 변수로 다시 집어넣어야 한다.

간단한 C++ 예제로 보면,

```cpp
#include <iostream>

void swap(int x, int y){
	int tmp = x;
	x = y;
	y = tmp;
}

int main(){
	int a = 10;
	int b = 20;

	swap(a, b);
	
	std::cout << a << " " << b << "\n"; # 10 20
}
```

main 함수 내에서 swap 함수에 a와 b를 전달하여 바꾸는 작업을 수행했지만, 결과에는 전혀 반영되지 않는다.

이유는 간단한데, 지역변수 및 매개변수(a,b)는 stack에 할당되기 때문이다. swap 함수를 호출하는 순간 x와 y가 할당되고 각 x,y에 **a,b 값이 복사**되어 사용된다. 그리고 **함수가 반환되면** 해당 함수의 스택 프레임이 **스택에서 제거된다.**  

<br>

# Call by reference

- 주소값을 전달

Call by reference도 함수의 인자값이 복사되어 들어간다는 점은 동일하다. 그러나 복사되는 값이 변수에 저장된 데이터가 아닌, 주소값이 복사된다.

```cpp
#include <iostream>

void swap(int *x, int *y){
	int tmp = *x;
	int *x = *y;
	*y = tmp;
}

int main(){
	int a = 10;
	int b = 20;

	swap(&a, &b);
	
	std::cout << a << " " << b << "\n"; # 20 10
}
```

함수에서 매개변수를 포인터 변수로 정의하고, 변수의 주소를 인자로 받으면, swap 함수 내에서 역참조 연산자를 통해 주소에 저장된 값에 직접 접근하여 a와 b의 값을 바꿀 수 있게 된다. 스택에는 각각 a와 b의 주소값을 저장하고 있는 포인터 변수가 들어가겠지?

<br>

# Call by assignment

자, 그래서 그럼 Call by assignment란 무엇이냐,

이건 python에서 사용하는 방식으로, 위의 값 복사와 주소값 참조와는 동작 방식이 다르다.

예를 들어 아래 두 개의 파일이 있다고 하자.

- a.py

```python
#main
from b import *

print("a :", id(VAL)) # a : 1234
func_b()
print("a :", id(VAL)) # a : 1234
print("VAL :", VAL) # VAL: 1
```

- b.py

```python
VAL = 1

def func_b():
	global VAL

	print("b: ", id(VAL)) # b : 1234
	VAL = 5
	print("b: ", id(VAL)) # b : 5678
	print("VAL :", VAL) # VAL : 5
```

a라는 파일에서 b라는 파일에 있는 func_b와 VAL을 import 해왔다. func_b에서는 global 변수인 VAL에 접근하여 값을 1에서 5로 바꾼다.

a라는 파일에서 func_b를 호출하면, a 파일에서 지칭하는 VAL과 b라는 파일에서 지칭하는 VAL이 동일하므로 func_b 함수 호출을 통해 VAL 값은 5로 수정되고, a 파일에서는 당연히 VAL이 5로 호출되어야 한다.  

**그런데 그렇지 않다.**  

id()는 변수의 주소값을 알려주는 함수이다.

func_b에서 VAL = 5로 할당하기 전, VAL 주소값은 1234로 a파일과 b파일 모두 출력값이 동일하다.

그런데 VAL = 5로 할당해주는 순간 b 파일에서 VAL의 주소값은 5678로 변경되고 VAL 값 또한 5로 변경된다. 함수 호출을 마치고 다시 a 파일에서 VAL의 주소값과 VAL값을 출력하면..? 여전히 주소값은 1234로, VAL 값은 1로 변경되지 않았다!

마치 b 파일에서 VAL = 5라고 할당해주는 순간 새로이 VAL이 생겨난 것 같다.

이유는 파이썬이 변수를 취급하는 방식 때문인데, 파이썬에서는 변수가 특정 메모리 공간을 할당해주는 개념보다 ‘객체에 붙여진 이름표’에 가깝게 동작한다.  

다음 코드를 파이썬에서 출력해보면,

```python
a = "hi"
b = "hi"
c = "hi"

print(id(a), " ", id(b), " " ,id(c)) # 1234 1234 1234
```

a,b,c 모두 같은 주소값을 가지는 것을 확인할 수 있다. “hi”라는 객체에 붙은 여러 개의 이름표가 변수 a,b,c인 것이다. 즉 “hi”라는 객체는 주소를 가지지만, 변수 a.b.c에 대해서는 위치를 고정적으로 지정해두지 않는다.  

다시 위 문제로 돌아가보자.

‘VAL = 5’라고 값을 할당하는 순간 VAL 변수는 5라는 객체를 나타내는 이름표가 되며, 해당 파일 내에서 VAL은 객체 5를 가리키는 변수 그 자체이므로, b 파일에서 만약 func_b() 함수를 실행 시킨 후 print(VAL)을 실행하면 5라고 출력할 것이다.

그러나 b 파일에서 func_b()를 실행시키지 않고, a 파일에서 func_b()를 호출한다면? func_b()를 호출하는 순간 b 파일에서 VAL은 객체 5를 나타내는 변수가 되어 VAL은 새로운 주소를 갖지만, a 파일에서는 import 할 때 가져온 VAL의 주소를 가지고 있으므로 바뀐 VAL의 주소는 알 수 없다. 따라서 수정전 VAL 값인 1이 출력된다.

그럼 한 개의 전역변수 데이터를 다른 파일에서 공유하여 조작할 수는 없는 거냐?

그렇지 않다. mutable 자료형(list, dictionary, set)은 가능하다.

VAL을 리스트 형태로 작성하여 아래와 같이 다시 작성해보면,

- a.py

```python
#main
from b import *

print("a :", id(VAL)) # a : 1234
func_b()
print("a :", id(VAL)) # a : 1234
print("VAL :", VAL) # VAL: []
```

- b.py

```python
VAL = []

def func_b():
	global VAL
	
	print("b: ", id(VAL)) # b : 1234
	val = [1,2]
	VAL = val
	print("b: ", id(VAL)) # b : 5678
	print("VAL :", VAL) # VAL : [1,2]
```

여전히 b 파일에서의 VAL값과 a 파일에서의 VAL값이 다른 것을 볼 수 있다.

이 역시 func_b() 내에서 **‘VAL = val’이라고 할당하는 순간, VAL은 객체 val을 지칭하는 새로운 변수**가 되기 때문이다. 

VAL 값을 수정하고 싶으면, extend나 append 함수 등을 사용하여 원래 전역변수 VAL에 저장되어 있는 객체 값 자체를 수정하면 된다.

이렇게.

- a.py

```python
#main
from b import *

print("a :", id(VAL)) # a : 1234
func_b()
print("a :", id(VAL)) # a : 1234
print("VAL :", VAL) # VAL: [1, 2]
```

- b.py

```python
VAL = []

def func_b():
	global VAL
	
	print("b: ", id(VAL)) # b : 1234
	val = [1,2]
	VAL.extend(val.copy())
	print("b: ", id(VAL)) # b : 1234
	print("VAL :", VAL) # VAL : [1,2]
```
