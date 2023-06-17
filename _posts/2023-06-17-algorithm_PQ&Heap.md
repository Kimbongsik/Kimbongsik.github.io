---
title: "[자료구조/C++] 우선순위 큐, 힙(Priority Queue, Heap))"
last_modified_at: 2023-06-17T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---

# Heap이란?(정의)

힙에 대해서 정의하려면, 먼저 알아야 하는 개념이 있다.

- Full Binary Tree
    - 모든 노드가 0개 또는 2개의 자식을 가지고 있음
    - leaf 노드를 제외한 모든 노드들이 2개의 chlid 노드를 가짐
    - 모든 leaf 노드의 개수: Internal Node 개수 + 1
- Perfect Binary Tree
    - 모든 노드가 2개의 자식을 가지고 있음
    - 모든 leaf 노드가 같은 레벨에 있음
    - 모든 노드의 개수 : 2^h + -1 (h(height)>0)
- **Complete Binary Tree(완전 이진 트리)**
    - 마지막 레벨을 제외한 나머지 레벨에 노드들이 모두 full
    - 노드 삽입 시, 최하단 왼쪽 노드부터 차례대로 삽입 된다
        - 즉, 힙에서 최하단 노드는 자식 노드를 하나만 가질 수 있으나 이 경우 right node만 가지는 건 불가능하다.
    - 이게 Heap

즉, 힙은 다음의 두 가지 조건을 만족해야 한다.

1. Shape property: “Complete Binary Tree” 형태여야 함
2. ORDER property: heap에 있는 **부모 노드는 자식 노드보다 크거나 같아야 한다. →max heap**

이러한 조건 때문에, 힙(max heap)에서 가장 큰 값은 최상위 노드, 즉 root 노드가 된다.

힙은 Complete binary tree이기 때문에, 중간에 비어 있는 노드가 없어야 하므로 linked list로 구현하는 것보다 array로 구현하는 게 훨씬 편하다. (포인터로 구현해도 되긴 함)

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/c58c643a-4720-4b4b-9265-2ad3d271f3a4)

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/8f0bb437-e881-4a99-b290-2db56b4e3a7c)

트리 순서대로, 루트는 리스트 0번에 저장.

루트 왼쪽 노드는 0 * 2 + 1 = 1번에 저장.

루트 오른쪽 노드는 0*2  + 2 = 2번에 저장.

level 3의 가장 왼쪽 노드는 1 * 2 + 1 = 3번에 저장.

level 3의 두 번째 노드는 1 * 2 + 2 = 4번에 저장.

..(반복)

# 구현

먼저 Heap의 큰 틀이다.

elements는 동적으로 할당 받는다. 

```cpp
template<class ItemType>
sturcture HeapType{
	void ReheapDown(int root, int bottom);
	void ReheapUp(int root, int bottom);

	ItemType* elements; //동적할당 될 array
	int numElements; //저장되어 있는 데이터의 개수
};
```

## ReheapDown & ReheapUp이란?

힙 구조에서 노드를 삭제한다거나 한다고 해보자. 이 때 힙의 구조가 깨질 수 있다. 이 때 다시 heap을 다시 구성할 때 필요한 기능이 ReheapDown과 ReheapUp이다. 

- **ReheapDown**
- 사용하는 경우: 루트가 자식 노드보다 값이 작음

![3](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/ea4484ce-a415-4ef6-a69e-b98484cb16fe)

위 그림의 경우, 맨 위 트리(E,H,I)가 힙 구조가 깨졌다. 이 때 다시 힙 구조를 만들어주기 위해서는, E의 자식 노드 중 가장 큰 노드와 바꾸어 줘야 한다. 

이 후, 아래에 있는 트리가 다시 힙 구조가 깨지므로, 힙이 만족되는 시점까지 값을 계속 내려보내준다. 

이를 ReheapDown이라고 한다.

- **ReheapUp**
- 사용하는 경우: bottom(마지막 레벨의 leftmost 노드)이 루트보다 값이 큼

![4](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/b67970d9-b6ef-4020-b521-db296ab9e766)

이번에는 최하단 노드의 가장 오른쪽 노드가 힙의 조건을 만족하지 않는 경우, 노드를 위로 올려서 힙의 조건을 만족시켜준다. 이를 ReheapUp이라고 한다. 해당 그림에서는 K가 새로 들어왔을 때, 힙에서 가장 큰 노드이므로 root까지 올려주는 작업을 수행하고 있다.

## ReheapDown 구현

```cpp
template<class ItemType>
void HeapType<ItemType>::ReheapDown(int root, int bottom){ 
//root: 루트 노드 때문에 힙이 깨진 경우이므로 root를 element로 받음
//bottom: 최말단 노드의 가장 오른쪽에 있는 노드. 즉 array의 length
	int maxChild;
	int rightChild;
	int leftChild;

	leftChild = root * 2 + 1;
	rightChild = root * 2 + 2;

	if(leftChild <= bottom){ //base case
		if(leftChild == bottom){ //노드가 하나밖에 없으므로 이를 root와 바꿈
			maxChild = leftChild;
		else{ //child가 2개가 있는 경우{
			if(elements[leftChild] <= elements[rightChild])
				maxChild = rightChild;
			else
				maxChild = leftChild;
		}
		if(elements[root] < elements[maxChild]){
			Swap(elements[root], elements[maxChild]);
			ReheapDown(maxChild, bottom); //maxChild는 바꾸기 전의 root
		}
	}
}
```

## ReheapUp

```cpp
template<class ItemType>
void HeapType<ItemType>::ReheapUp(int root, int bottom){
	int parent;

	if (bottom > root){ //
		parent = (bottom - 1) / 2; //공식
		if(elements[parent] < elements[bottom]){
			Swap(elements[parent], elements[bottom]);
			ReheapUp(root, parent);
		}
	}
}
```

# 우선순위 큐(Priority Queue)란? (정의)

우선순위 큐란 우선순위가 가장 높은 요소에 어느 순간이든 접근 가능하도록 하는 구조를 말한다. 즉, Unsorted list든, Sorted list든, Linked로 구현하든 상관이 없지만, 우리는 위에서 Heap을 배웠다. Heap은 어떤 특징을 가지는가? 바로 가장 큰 요소(우선순위가 가장 높은 노드)는 항상 최상위 노드에 위치한다. 그래서 주로 힙으로 구현한다.

- Linked Sorted List → Enqueue 시간복잡도: O(N)
- Binary Search Tree → Enqueue/Dequeue시 평균 시간복잡도: O(lgN)
    - 그러나 worst case의 경우, O(N)
- Heap → O(lgN). worst case에서도 O(lgN)을 보장함

위 힙에서 ReheapDown과 ReheapUp 사용법을 익혔으니, 우선순위 큐 구현은 더욱 쉽다.

# 구현

- 선언

```cpp
class FullPQ(){};
class EmptyPQ(){};

template<class ItemType>
class PQType{
public:
	PQType(int);
	~PQType();
	void MakeEmpty();
	bool IsEmpty() const;
	bool IsFull() const;
	void Enqueue(ItemType newItem);
	void Dequeue(ItemType& item);
private:
	int length;
	HeapType<ItemType> items; //Heap에서 구현한 클래스 사용
	int maxItems;
};
	
```

- 정의

```cpp
template<class ItemType>
PQType<ItemType>::PQType(int max){
	maxItem = max;
	items.elements = new ItemType[max]; //최대값 만큼 할당
	length = 0;
}

template<class ItemType>
PQType<ItemType>::~PQType(){
	delete [] items.elements;
}

template<class ItemType>
void PQType<ItemType>::MakeEmpty(){
	length = 0;
}

template<class ItemType>
void PQType<ItemType>::Dequeue(ItemType& item){ //Dequeue한 아이템을 item 변수에 담음
	if(length == 0)
		throw EmptyPQ();
	else{
		item = items.elements[0]; //최상위 노드
		items.elements[0] = items.elements[length-1]; //bottom 노드의 값을 맨 위에 넣어주고
		length--; //Dequeue이므로 길이 1 감소. 즉 리스트 맨 마지막 노드는 사라진다(리스트에 값으로 존재하나, 의미 없는 값 처리)
		items.ReheapDown(0, length-1); //해당 노드를 ReheapDown으로 내려준다 (내려가면서 정렬된다)
	}
}

template<class ItemType>
void PQType<ItemType>::Enqueue(ItemType newItem){
	if(length == maxItems){
		throw FullPQ();
	else{
		length++; //길이 하나 늘려주고
		items.elements[length-1] = newItem; //리스트의 맨 뒤에 값을 넣어준뒤 
		items.ReheapUp(0, length-1); //ReheapUp으로 올려준다.
	}
}
```

즉, 정리하면

- **Priority Queue Enqueue:**
- **Priority Queue Dequeue**
    - 최상위 노드 빼고
    - 최상위 노드에 가장 아래의 노드를 넣어주고
    - ReheapDown으로 정렬

# 시간 복잡도 비교

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ce55626-0928-4847-8c71-b7e964d17e41/Untitled.png)
