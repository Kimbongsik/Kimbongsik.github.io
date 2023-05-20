---
title: "[자료구조/C++] Unsorted & Sorted List(비정렬&정렬 리스트) 구현 방법"
last_modified_at: 2023-04-27T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---

# Unsorted List(비정렬 리스트)

## 정의

- 말그대로 정렬되어 있지 않은 리스트
- 새로운 원소 삽입 시, 리스트의 맨 뒤나 맨 앞에 삽입해주면 된다

## 기능

```cpp
#include "ItemType.h"

class UnsortedType{
public:
		void UnsortedType(); //생성자
		bool IsFull() const; //리스트가 모두 찼는지
		int LengthIs(); const; //길이 반환 
		void RetrieveItem(ItemType& item, bool& found); //아이템 탐색
		void InsertItem(ItemType item); //아이템 삽입
		void DeleteItem(ItemType item); //아이템 삭제 
		void ResetList(); //GetNextItem 사용 시, 리스트의 맨 앞에서 시작
		void GetNextItem(ItemType& item); //다음 아이템
private:
		int length; //리스트 원소 개수
		ItemType info[MAX]; //아이템을 담을 리스트
		int currentPos; //현재 위치
}
```

## 구현

```cpp
//ItemType.h
#include <fstream>
const int MAX = 5;
enum RelationType{LESS,GREATER,EQUAL};

class ItemType{
public:
	ItemType();
	RealationType ComparedTo(ItemType) const;
	void Print(std::ostream&) const;
	void Initialize(int number);
private:
	int value;
}
```

```cpp
//ItemType.cpp

#include <fstream>
#include <iostream>
#include "ItemType.h"

ItemType::ItemType(){ value = 0;}

void ItemType::Initialize(int number){
	value = number;
}

RelationType ItemType::ComparedTo(ItemType) const{
	if (value < otherItem.value)
		return LESS;
	else if (value == otherItem.value)
		return EQUAL;
	else
		return GREATER;
}

void ItemType::Print(std::ostream& out) const
{
	out << value;
}
```

```cpp
//UnsortedType.h

#include "ItemType.h"

void UnsortedType::UnsortedType(){
	length = 0; //원소 개수 0으로 초기화
}

bool UnsortedType::IsFull(){
	return(length == MAX);
}

int UnsortedType::LenghtIs(){
	return length; //private 데이터인 length의 값을 반환
}

void UnsortedType::RetrieveItem(ItemType& item, bool& found){ //binary search
	bool moreToSearch; //location이 length를 넘어가지 않았는지 검사
	int location = 0;
	
	found = false;
	moreToSearch = (location <length);
	while(!found && moreToSearch){
		switch(item.ComparedTo(info[location])){
		case LESS:
		case GREATER: location++; moreToSearch = (location<length); break;
		case EQUAL: found = true; item = info[location]; break;
		}
	}
}

void UnsortedType::InsertItem(ItemType item)
{
	info[length] = item;
	length++;
}

void UnsortedType::DeleteItem(ItemType item){
	int location = 0;
	while(item.ComparedTo(info[location]) != EQUAL){
		location++;
	}
	info[location] = info[length-1]; //비정렬 리스트이므로, 리스트의 맨 뒤 원소를 끌어다 넣어주면 된다
	length--;
}

void UnsortedType::ResetList(){
	currentPos = -1;
}

void UnsortedType::GetNextItem(ItemType& item){
	currentPos++;
	item = info[currentPos];
}

```

</br>

# Sorted List(정렬 리스트)

## 정의

- 말그대로 정렬되어 있지 않은 리스트
- 새로운 원소 삽입 시, 리스트의 맨 뒤나 맨 앞에 삽입해주면 된다
- 특정 원소 삭제 시, 삭제한 원소 뒤의 원소들을 앞으로 당겨줘야 한다

원소의 삽입, 삭제를 제외하고 Sorted List는 Unsorted List와 유사하다. 

## 구현

```cpp
//sorted.cpp

#include "ItemType.h"

void SortedType::InsertItem(ItemType item){
	bool moreToSearch;
	int location = 0;

	moreToSearch = (location < length); //length보다 작을 때까지 search
	while(moreToSearch){
		switch(item.ComparedTo(info[location]){
		case LESS: moreToSearch = false; break;
		case GREATER: location++; moreToSearch = (location < length); break;
	}

	for(int index = length; index > location; index--){
		info[index] = info[index-1]; } //location보다 뒤에 있는 원소들을 한 칸씩 "뒤로" 당겨준다.
info[location] = item; //location 자리에 삽입
length++;
}

void SortedType::DeleteItem(ItemType item){
	int location = 0;
	while(item.ComparedTo(info[location]) != EQUAL)
		location++; //리스트의 원소와 삭제할 아이템이 일치하지 할 때까지 location 추가

	for(int index = location + 1; index < length; index++){
		info[index - 1] = info[index]; //삭제할 아이템의 뒤에 있는 원소들을 한 칸씩 "앞으로" 당겨준다.
	length--; //아이템을 하나 제거했으므로, len 감소
```
