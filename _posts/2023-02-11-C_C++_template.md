---
title: "[C/C++] 템플릿(template) 사용법"
last_modified_at: 2023-02-11T16:20:02-05:00
categories:
  - C_C++

tags:
  - C_C++
---

템플릿은 함수나 클래스를 개별적으로 다시 작성하지 않아도 여러 자료형으로 사용할 수 있도록 만들어 둔 틀이라고 보면 된다. Stack 구조의 경우 data가 homogeneous하기 때문에 데이터의 타입이 여러가지라면 타입 별로 스택을 하나하나 구현해야 한다. 이 때 template을 사용하면 좋다.

템플릿은 크게 두 가지 유형이 있다.
<br/>

- 함수 템플릿(function template)
- 클래스 템플릿(class template)  

<br/>
# 함수 템플릿(function template)

함수를 만들어 낼 때 함수의 기능은 명확하지만 자료형을 모호하게 둔다. 

예를 들어 인자 2개를 받아 더한 값을 return하는 함수를 보면,

```cpp
int sum(int a, int b)
{
	return a+b;
}
double sum(double a, double b)
{
	return a+b;
}
```

이런 식으로 서로 다른 타입을 계산하기 위해 수행 기능이 동일한데도 불구하고 함수를 두 개 만들어줘야 했는데, 템플릿을 사용하면 하나의 함수만 정의하면 된다. 이렇게..

```cpp
template <typename T>
T sum (T a, T b)
{
	return a+b;
}

int main(void)
{
	int a = 1, b = 2;
	double d1 = 1.1, d2 = 2.2;
	string s1= "hi", s2 ="bye";
	cout << sum<int>(a,b) << endl;
	cout << sum<double>(d1,d2) << endl;
	cout << sum<string>(s1,s2) << endl;

	return 0;
}
```
<br/>
main 함수에서 template 함수를 호출할 때 < > 안에 어떤 자료형을 쓸 건지 표시하여 호출하면 된다. 이렇게 편할 수가..

이번엔 인자를 두 개 받을 때 두 개의 타입이 다른 경우를 보자. 

이 경우에는 함수 이름 뒤에 <> 안에 명확하게 타입을 밝히지 않고, 컴파일러가 스스로 자료형을 판단한다. 

```cpp
#include <iostream>
#include <string>

using namespace std;

template <class T1, class T2>
void printAll(T1 a, T2 b)
{
	cout << a << endl;
	cout << b << endl;
};

int main(void)
{
	string s1 = "hi", s2 = "bye";
	int num1 = 1, num2 = 2;
	double d1 = 1.1, d2 = 2.2;

	printAll(s1,s2);
	printAll(num1, num2);
	printAll(d1,d2);
	
	return 0;
}
```
<br/>
참고로 <typename T>와 <class T>는 정확히 같은 의미를 갖는다. 그러나 typename을 쓰는 것이 권장된다고 한다. 왜냐하면 class T라고 하면 T에 뭔가 class를 써야할 것처럼 느낄 수 있는 혼동을 방지하기 위해..C++ 위원회에서 typename이라는 이름을 사용하자고 정했다고 한다. 
<br/>
# 클래스 템플릿(class template)

함수 템플릿과 동작은 같으나 대상이 함수가 아닌 클래스이다. 클래스 템플릿을 만들면 타입에 따라 다르게 동작하는 클래스 집합을 만들 수 있다. 함수 템플릿과 차이점이 있다면 함수 템플릿과 다르게 **클래스 템플릿은 반드시 템플릿 인수를 명시해줘야 한다.** 클래스 타입의 데이터 타입이 결정되기 위해서는 생성자가 호출되어야 하기 때문에, 명시적으로 템플릿 인수를 작성하지 않으면 어떤 타입에 대한 메모리를 할당해야 하는지 몰라 객체를 생성할 수 없게 된다. 

 ****

 함수 템플릿과 마찬가지로

```cpp
template <typename T>
class Tname
{
	//class member
}
```

이런 식으로 사용한다. 

```cpp
template <typename T>
class Data
{
	T val;
public:
	Data(T v) :val(v)
	{};
	T Getval();
	void printall()
	{
		cout << "멤버 변수: " << val << endl;
	}
};

template<typenamve T>
T Data<T>::Getval() //명시
{
	return val;
} 

int main()
{
	Data<int> t_int(5);
	Data<string> t_str("hi");

	t_int.printall();
	t_str.printall();

	cout << typeid(t_int.Getval()).name() << endl;
	cout << typeid(t_str.Getval()).name() << endl;
}
```
<br/>
출력 결과)

멤버변수: 5

멤버변수: hi

int

class std::basic_string<char, struct std::char_traits<char>, class std::allocator<char> >
