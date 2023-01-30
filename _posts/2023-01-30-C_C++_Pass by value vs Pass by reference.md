---
title: "[C/C++] Pass bye value vs Pass by reference"
last_modified_at: 2023-01-30T16:20:02-05:00
categories:
  - C_C++

tags:
  - C/C++
---

Pass by values는 값에 의한 전달, Pass by reference는 참조에 의한 전달이다.

## Pass by value(값에 의한 전달)

지역변수(Local variable)에서 값을 복사해 복사본을 함수에 넣어주는 것을 의미. 

```cpp
int passByValue(int x);

int main()
{
	int num = 10;
	passByValue(num);
retrun 0;
}

int passByValue(int x)
{
	return x;
}
```

PassByValue 함수를 불러올 때 main함수에서 num값을 복사하여 넘겨주고, 이 값은 복사본 num으로 함수에서 사용된다. 따라서 다른 함수에서 값을 변경한다고 해도 num의 값은 변하지 않는다. 

: Pass by value

## Pass by reference(참조에 의한 전달)

패스 바이 레퍼런스는 값이 저장된 주소를 보내줌. pass by value는 변수의 값을 복사해 보내줬지만, pass by reference는 변수가 저장된 주소값을 부내주기 때문에 직접적으로 변수에 접근이 가능함.

```cpp
void swaping(int&x , int& y)
{
	int num = x;
	x = y;
  y = num;
}

int main()
{
	int num1 = 1;
	int num2 = 2;
	cout << "num1: " << num1 << " num2: " << num2 << endl; //num1: 1 num2: 2
  swaping(num1, num2);
	cout << "num1: " << num1 << " num2: " << num2 << endl;
	return 0; // num1: 2 num2: 1
}
```

swaping 함수에서 파라미터는 각각 int& x, int& y이다. 이는 변수에 직접적으로 접근하는 방식으로, num1의 별명은 int x, num2의 별명은 int y가 되기 때문에 swaping 함수 호출 이후 실제 num1과 num2의 값은 뒤바뀌게 된다. 




<aside>
💡 참조 사이트: [https://issac-min.tistory.com/34]

</aside>
