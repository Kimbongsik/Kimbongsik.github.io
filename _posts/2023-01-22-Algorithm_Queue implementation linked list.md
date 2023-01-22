---
title: "[자료구조] 큐(Queue) Linked list로 구현"
last_modified_at: 2023-01-22T16:20:02-05:00
categories:
  - algorithm

tags:
  - 자료구조
---

Operation

- MakeEmpty
- IsEmpty
- IsFull
- Enqueue (ItemType newItem)
- Dequeue(ItemType& item

```cpp
#include "ItemType.h"

template<class ItemType>
class QueType
{
public:
	QueType();
	~QueType();
	bool IsEmpty() const;
	void IsFull() const;
	void Enqueue(ItemType item);
	void Dequeue(ITemType& item);
	void MakeEmpty();

private:
	NodeType<ItemType>* qFront;
	NodeType<ItemType>* qRear;
};

template<class ItemType>
QueType<ItemType>::QueType() //constructor
{
	qFront = NULL;
	qRear = NULL;
}

template<class ItemType>
bool QueType<ItemType>::IsEmpty() const
{
	return (qFront == NULL)
}

template<class ItemType>
void QueType<ItemType>::Enqueue(ItemType newItem)
{
	NodeType<ItemType>* ptr;
	ptr = new NodeType<ItemType>;
  ptr -> info = newItem;
	ptr -> next = NULL;
	if (qRear NULL)
		qFront = ptr;
	else
		qRear-> next = ptr;
	qRear = ptr;
}

template<class ItemType>
void QueType<ItemType>::Dequeue(ItemType item)
{
	tempPtr = qFront;
	item = qFront -> info;
	qFront = qFront -> next;
	if (qFront == NULL)
		qRear = NULL; //dangling pointer(qR가 delete 후 존재하지 않는 곳을 가리키고 있을) 가능성 제거
	delete tempPtr;
}
```