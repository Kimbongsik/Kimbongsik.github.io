---
title: "[자료구조] 단일 연결 리스트(Linked List) 구현"
last_modified_at: 2023-04-13T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - 코드 트리
  - algorithm
---


리스트를 구현하는 방법으로 두 가지가 있는데, 하나는 배열을 이용하는 것이고 다른 하나는 동적할당을 이용해 구현하는 것이다. 이 중에서도 동적할당 구현을 살펴본다. 

동적 할당은 새로운 자료가 들어올 때마다 새로운 주소로 메모리를 할당해주어 한정적인 데이터만 넣을 수 있는 배열보다 더 유동적으로 메모리를 관리할 수 있다. **연결 리스트란, 노드가 다음 순서의 노드의 위치정보를 가지고 있는 방식으로 데이터를 저장하는 자료구조이다.** 

연결 리스트의 원소는 ‘노드’라고 표현하며, 이 노드는 값과 포인터로 이루어져 있다. 일반적으로 첫 번째 노드는 ‘head’라고 하며 마지막 노드는 ‘tail’이라고 표현한다. 

크게 세 가지 종류가 있다. 

**-단일 연결 리스트 (singly linked list)**

각 노드에 데이터 공간과 한 개의 포인터를 가지고 있어 노드의 포인터가 다음 노드를 가리키는 경우. 따라서 특정 노드의 후행 노드는 쉽게 접근할 수 있으나 선행 노드에 대한 접근은 헤드 노드부터 재검색 해야 한다는 단점이 있다. 

![_1](https://user-images.githubusercontent.com/63995044/231798988-65ce2fd5-10da-4f2a-bd03-ad06c2cbf2b7.png)

**-이중 연결 리스트 (doubly linked list)**

각 노드가 두 개의 포인터를 가짐. 포인터는 각각 다음 노드와 이전 노드를 가리킴.

![2](https://user-images.githubusercontent.com/63995044/231799003-1dec8d0b-0248-454a-a54f-0c1986fffdd6.png)

**-원형 연결 리스트 (circular linked list)**

단일 연결 리스트에서 마지막 노드가 처음 노드를 가리켜 원형 형태를 띄는 연결리스트. 

![3](https://user-images.githubusercontent.com/63995044/231799005-7f14d85b-624e-4e1f-94b5-f3732f76c5ed.png)

이중 연결 리스트에서 만들면 이중 원형 연결 리스트.

![4](https://user-images.githubusercontent.com/63995044/231799008-f135ac02-f2a2-4182-a6bd-d35a4ac5ee78.png)

밑에는 구현.

- 단일 연결 리스트

```cpp
class StringNode 
{
	private:
		string elem; //element
		StringNode* next; //다음 노드를 가리킬 포인터

		friend class StringLinkedList;
};

class StringLinkedList
{
	public:
		StringLinkedList();
		~StringLinkedList(); //소멸자
		bool empty()const; //리스트가 비었는지 확인
		const string&front()const; //첫번째 원소 return
		void addFront(const string&e); //맨 앞에 원소 추가
		void removeFront(); //맨 앞 원소 제거
	private:
		StringNode* head;
};

//생성자
StringLinkedList::StringLinkedList()
		:head(NULL){} // head==NULL이면 빈 리스트
//소멸자
StringLinkedList::~StringLinkedList()
{
	while (!empty()) removeFront();
} //empty 함수가 true를 리턴할 때까지 removeFront 함수를 실행.

bool StringLinkedList::empty const
{
		return head == NULL;
} //head==NULL이면 빈 리스트이므로.

const string& StringLinkedList::front() const
{
		return head->elem;
}

//맨 앞 원소 추가.
/* 단계 1: 새로운 노드 생성
   단계 2: 새로운 노드의 포인터가 현재 head 값을 가리킴.
   단계 3: head 변수가 새로 생성한 노드를 가리킴. 
*/
void StringLinkedList::addFront(const string& e)
{
		StringNode* v = new StringNode; // 단계 1
		v->elem = e; //새로운 노드에 값 추가
		v->next = head; // 단계 2
		head = v; //단계 3
}

//맨 앞 원소 삭제.
/* 단계 1: head의 값을 임시 변수(일반적으로 old를 사용)에 저장.
	 단계 2: head의 값을 old의 다음 노드로 변경(head가 두 번째 노드를 가리킴.)
	 단계 3: old 삭제.
*/

void StringLinkedList::removeFront() 
{
		StringNode* old = head; // 단계 1
		head = old -> next; // 단계 2
		delete old; // 단계 3
} 

```

연결 리스트의 경우 원소를 추가, 삭제하는 작업의 시간 복잡도는 O(1)이고 원소에 접근하는 경우의 시간 복잡도는 O(n)이기 때문에, **특정 원소에 접근 하는 경우에는 배열을, 원소의 추가 삭제가 필요한 경우는 연결리스트를** 이용하는 것이 효율적이라고 한다. 

참고 사이트:

[https://sectumsempra.tistory.com/56](https://sectumsempra.tistory.com/56)

[https://huiyu.tistory.com/154](https://huiyu.tistory.com/154)
