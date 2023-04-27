---
title: "[자료구조] 큐(Queue) 정의 및 구현(+문제점 해결)"
last_modified_at: 2023-04-27T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---

# 큐(Queue)란?

## 정의

- 맥도날드 줄 서기 같은 것
- 먼저 들어온 아이템이 먼저 나가는, 선입선출(First In, First Out)

## 기능

- Enqueue(아이템 삽입)
- Dequeue(아이템 빼기)

---

# 문제점 1

오늘은 좀 더 기본적인 Queue에 대해 정리하려고 한다. 큐(Queue)는 선입선출 방식의 자료구조로, 가장 베이직 한 자료구조 중 하나이다. 아이템을 선형적으로 저장하면 Linear Queue, 원형으로 저장하면 Circular Queue라고 한다. 그럼 원형큐는 왜 쓸까?

Enqueue와 Dequeue가 반복적으로 발생하는 상황을 생각해보자. 

![1](https://user-images.githubusercontent.com/63995044/234858921-15d0db9a-2de6-4b35-8308-e7309f58a40a.png)

Queue는 front와 rear 포인터를 통해 Enqueue와 Dequeue를 수행한다. front 포인터는 데이터의 맨 앞을, rear는 새로 들어온 데이터를 가리킨다. Dequeue를 할 때, 배열 안에 있는 데이터가 사라지는 것은 아니고, 이 포인터의 이동으로만 이루어지기 때문에, **front와 rear 포인터가 배열의 맨 끝에 다다르면 더이상 데이터를 추가할 수 없게 된다.**

뿐만 아니라 front의 앞 배열의 공간은 그대로 할당 되어 있는데, 데이터를 추가하겠다고 일일이 배열의 크기를 늘려줄 수도 없는 노릇이다. 

그렇다면 한정된 배열의 자원에서 큐라는 구조를 유지하려면?

![2](https://user-images.githubusercontent.com/63995044/234858928-d78afb81-28d2-4fe6-be99-5d7650055d63.png)

**답은, “원형 형태로 돌리면 된다.”**

```cpp
if(rear == maxQueue -1) //rear이 배열의 맨 끝에 다다르면, 
	rear = 0; //배열의 맨 앞으로 간다
else
	rear += 1; //그렇지 않으면 전진
```

 더 쉽게 짜는 방법도 있다. 

```cpp
rear = (rear + 1) % maxQueue; 
```

# 문제점2

자료구조를 구현할 때, 항상 들어가는 기능 중 ‘IsFull(새로운 아이템을 추가할 수 있는가?)’과, ‘IsEmpty(아이템을 더 삭제할 수 있는가?)’가 있다. 해당 기능 없이 함부로 데이터의 삽입/삭제가 일어나면 메모리 영역을 침범하여 에러가 발생하기 십상이기 때문이다. 그럼 여기서 큐의 IsFull과 IsEmpty는 어떻게 구현할 것인가?

**IsFull**

![3](https://user-images.githubusercontent.com/63995044/234858930-6922d3f6-2728-4d51-9b7f-8656e564ae65.png)

arr[2]에 5, arr[3]에 10, arr[0]에 20이 들어있는 상황에서 arr[1]에 30이 들어왔다(원형큐라는 걸 눈치채셨길!). rear + 1 == front이므로, 더이상 데이터를 추가할 수 없다.

**IsEmpty**

![4](https://user-images.githubusercontent.com/63995044/234858933-27ae7f73-70fa-40f0-985b-7364b2c8f4c0.png)

이번엔 Deuque를 통해 배열의 모든 아이템을 빼냈다. 어? 그런데 rear + 1 == front로, IsFull의 상태와 동일하다!

그렇다. 해당 원형큐에서는 IsFull과 IsEmpty를 구분할 수 없는 큰 문제가 있다.

해결 방법은? 공간 하나를 버리면 된다.

front == rear일 때를 Empty 상태로 만들기 위해 배열의 한 칸을 비워둔다. 

**IsFull**

![5](https://user-images.githubusercontent.com/63995044/234858936-23487504-d571-4c66-8270-086a95fceb17.png)

이렇게. Enqueue를 한들, rear는 front에 저장된 공간을 덮어쓸 수 없다.

즉, `if(IsFull()==true){rear + 1 = front)};`

**IsEmpty**

![6](https://user-images.githubusercontent.com/63995044/234858939-9f248d1c-598d-469d-8063-ba8df4bb1cdf.png)

초기상태와 동일하게, front와 rear가 같은 공간을 가리키고 있다면, Empty 상태가 된다. 

즉, `if(IsEmpty()==true){rear == front)};`

# 구현

## 초기화

![7](https://user-images.githubusercontent.com/63995044/234858940-4bec5cdb-fd5d-4c92-9812-47e3d67c336c.png)

```cpp
QueType(int max){
	maxQueue = max + 1;
	front = maxQueue - 1; //배열의 맨 끝에서 시작
	rear = maxQueue - 1; //배열의 맨 끝에서 시작
	items = new ItemType[maxQueue];
}
```

## Enqueue & Dequeue

```cpp
Enqueue(ItemType newItem){
	rear = (rear+1)%maxQueue;
	items[rear] = newItem;
}

Dequeue(ItemType &item){
	front = (front+1)%maxQueue;
	item = items[front];
}
```

## 전체 코드

```cpp
#include "ItemType.h"

template<class ItemType>
class QueType
{
	public:
		QueType();
		QueType(int max);
		~QueType();
		bool IsEmpty();
		bool IsFull() const;
		void Enqueue(ItemType newItem);
		void Dequeue(ItemType& item); //변수 item에 Dequeue한 아이템 저장
	private:
		int front;
		int rear;
		int maxQueue;
		ItemType* items;
}

template<class ItemType>
QueType<ItemType>::QueType(int max){
	maxQue = max + 1;
	front = maxQueue - 1;
	rear = maxQueue - 1;
	items = new ItemType[maxQueue];
}

template<class ItemType>
QueType<ItemType>::~QueType(){
	delete [] items;
}

template<class ItemType>
bool QueType<ItemType>::IsEmpty(){
	return (rear == front);
}

template<class ItemType>
bool QueType<ItemType>::IsFull(){
	return ((rear+1) % maxQueue == front);
}

template<class ItemType>
void QueType<ItemType>::Enqueue(ItemType newItem){
	rear = (rear+1) % maxQueue;
	items[rear] = newItem;
}

tempalte<class ItemType>
void QueType<ItemType>::Dequeue(ItemType newItem){
	front = (front+1) % maxQueue;
	item = items[front];
}
```