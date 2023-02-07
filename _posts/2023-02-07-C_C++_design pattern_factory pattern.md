---
title: "[디자인패턴] 팩토리 패턴(factory pattern)"
last_modified_at: 2023-02-07T16:20:02-05:00
categories:
  - C_C++

tags:
  - C_C++
---


코드가 목적에 맞게 굴러가는 것을 코딩 우선순위의 1순위로 둘 수 있지만, 훗날 코드의 유지 및 관리를 위해 “좋은 코드”를 작성하는 것 또한 매우매우 중요하다. 요즘은 객체지향프로그래밍을 연습해보고 있어서, 저명한 디자인 패턴에 대해 정리한다. 

## 팩토리 패턴

---

> 결론부터 말하자면, 팩토리 패턴은 부모 클래스에서 자식 클래스가 어떤 객체를 생성할지 결정하도록 하는 패턴이다.
> 

가상으로 커피숍을 운영해보면서 확인해보자. Coffee 클래스를 부모 클래스로 하고, 

```cpp
class Coffee{
	virtual void NewCoffee(char* type) = 0;
};

class Latte: public Coffee{
	void NewCoffee(char* type){
		LatteCoffee* latte = new LatteCoffee();
		latte->open(type);
	}
};

class Americano: public Coffee{
	void NewCoffee(char* type){
		AmericanoCoffee* americano = new AmericanoCoffee();
		americano->open(type);
	}
};
```

손님이 주문한 커피의 종류에 따라 새로운 클래스들의 인스턴스를 생성하게 되는데, 이렇게 코드를 작성하게 되면, 나중에 카페에 새로운 메뉴가 생기거나, 메뉴에 변경 사항이 생기게 됐을 경우 코드를 다시 확인하고 추가 또는 제거해야 한다는 번거로움이 생긴다. 카페가 크게 번성하여 관리해야 할 메뉴가 말도 안 되게 많아진다면, 해당 방식을 사용하는 것은 너무도 비효율적이다.

이를 해결하기 위해, NewCoffee를 상위 클래스에서 처리해주면 된다. 

```cpp
class CoffeeType {};
class Latte : public CoffeeType {};
class Americano : public CoffeeType {};
class Espresso : public CoffeeType {};

class Coffee{
	virtual CoffeeType* createCoffee(char* type){
	// ~(생략) 
	return new Latte;
	// ~
	return new americano;
	// ~ 
	return new espresso;
	}
	void NewCoffee(char* type){
		CoffeeType* coffee = createCoffee(type);
		coffee->open();
	}
};

class Latte : public Coffee {};
class Americano : public Coffee {};
class Espresso : public Coffee {};

int main(){
	Latte latte;
	latte.NewCoffee("latte");
}
	
```

<aside>
💡 참고 사이트: [https://1d1cblog.tistory.com/394](https://1d1cblog.tistory.com/394)

</aside>