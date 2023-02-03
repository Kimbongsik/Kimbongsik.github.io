---
title: "[C/C++] 접근 지정자(access modifier)"
last_modified_at: 2023-01-16T16:20:02-05:00
categories:
  - C_C++

tags:
  - C_C++
---

구조체는 모든 멤버가 기본적으로 public멤버로 작동한다.

그래서 구조체를 선언한 후, 멤버를 초기화 하기 위해 직접 접근이 가능하다. 

```cpp
struct DateStruct{
.. //생략
}

int main()
{
	DateStruct birthDay;
	birthDay.month = 3;
	birthDay.day = 13;

	return 0;
}
```

이 때 ‘public’ 멤버는 구조체 또는 클래스 외부에서 접근할 수 있는 구조체/클래스의 공개 멤버이다. main() 함수가 구조체 외부에 있음에도 불구하고 구조체 DateStruct의 멤버에 직접 접근한 이유가 바로 여기에 있다.

만약 이 코드가 struct가 아닌 class일 경우에는 오류가 뜨게 된다. 클래스는 기본적으로 모든 구성원이 private이기 때문이다. private 멤버는 오직 클래스의 다른 멤버만 접근할 수 있는 비공개 멤버이다. 따라서 main() 함수가 외부에 있으므로 DateClass의 멤버가 아니게 되기 때문에 클래스의 멤버에 접근할 수 없다. 그래서 외부에서 접근하기 위해서는 ‘public’멤버를 따로 사용해줘야 한다.

 

```cpp
class DateClass
{
	public:
		int m_month;
		.. //생략
}

int main()
{
	DateClass birthDay;
  birthDay.m_month = 3;
	.. //생략

return 0;
}
```

public과 같은 것을 ***접근 지정자***라고 한다. 

- public : 공개 멤버. .클래스 외부에서 접근 가능
- private : 비공개 멤버, 클래스 내에서만 접근 가능
