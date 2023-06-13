---
title: "[자료구조/C++] 정렬 알고리즘(정의, 시간 복잡도, 구현까지)"
last_modified_at: 2023-06-13T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---

# Selection Sort

- 정의 : 정렬된 부분과 그렇지 않은 부분으로 나누고, unsorted 영역에서 가장 작은 elements를 sorted 영역으로 올림

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/445ec7c3-03a7-4c44-8384-164c646da18d)

- 시간 복잡도 계산
    - Sum = (N-1) + (N-2) + (N-1) … + 2 + 1
    - Sum = N * (N-1)
    - Sum = **O(N^2)**
- 구현
    
    ```cpp
    template<class ItemType>
    int MinIndex(ItemType values[], int start, int end){
    	int indexOfMin= start;
    	for(int index = start + 1; index <= end; i++)
    		if(values[index] < values[indexOfMin])
    				indexOfMin = index;
    
    	return indexOfMin;
    }
    
    template<class ItemType>
    void SelectionSort(ItemType values[], int numValues){
    	int endIndex = numValues - 1; //numValues: value의 개수
    	for(int current = 0; current <endIndex; current++)
    		Swap(values[current], values[MinIndex(values, current, endInedx)]);
    }
    ```
    

# Bubble Sort

- 정의: 끝에서 시작하여 이웃한 두 인덱스의 위치를 바꿈. 가장 작은 요소가 맨 위로 “bubble up” 됨
- 시간 복잡도: **항상 O(N^2)**
- 구현

```cpp
template<class ItemType>
void BubbleUp(ItemType values[], int start, end){ //인접한
	for(int index = end; index > start; index--){
		if(values[index] < values[index-1])
			Swap(values[index], values[index-1]);
	}
}

template<class ItemType>
void BubbleSort(ItemType values[], int numValues){
	int current = 0;
	while(curruent < numValues - 1){
		BubbleUp(values, current, numValues - 1)
		current++; //가장 작은 값이 BubbleUp을 하며 맨 위로 올라가므로, current는 항상 정렬된 인덱스이다.
							//따라서 하나씩 더해 정렬되지 않은 부분부터 정렬하도록 하게 한다.
	}
}
```

# Insertion Sort

- 정의: 새로운 카드를 삽입하는 것 같은 알고리즘. count를 하나씩 늘려나가면서 그 count만큼의 인덱스에서 계속 정렬하는 걸 반복하는 거다. 새로운 카드가 들어오면 또 전체 카드 정렬하고, 또 추가되면 또 전체 정렬하는 알고리즘.

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/92eaf4fb-3e6c-49cf-8572-77432552108b)

(그림이 좀 구리지만.. 대강 이런 식. 앞에는 이미 정렬된 상태가 된다.)

- 시간복잡도: O(N^2)
- 구현

```cpp
template<class ItemType>
void InsertItem(ItemType values[], int start, int end){
	bool finished = false;
	int current = end;
	bool moreToSearch = (current != start);

	while(moreToSearch && !finished){
		if(values[current] < values[current - 1]){
			Swap(values[current], values[current-1]);
			current--;
			moreToSearch = (current != start);
		}
		else
			finished = true;
}

template<class ItemType>
void InsertionSort(ItemType values[], int numValues){
	for (int count = 0; count < numValues; count++)
		InsertItem(values, 0, count);
	}
}
```

# Heap Sort

- 정의: 힙의 성질을 이용한 정렬. root 노드는 항상 가장 큰 값이므로, 리스트의 맨 뒤에 있는 값과 루트 노드를 계속 바꾸어 정렬하는 방법. 리스트의 하단은 이미 완벽히 정렬되어 있으므로, 아래를 고려할 필요가 없어진다.
- 시간 복잡도: Reheap Down 과정에서 각 요소는 자신의 두 자식과 비교되고(더 큰 값과 교환). 하지만 각 레벨에서는 단 한 요소만이 이 비교를 수행하며, N개의 노드를 가진 완전 이진 트리는 O(log2N)개의 레벨만 가진다.
    - (N/2) * O(log N) : origin heap 생성
    - (N-1) * O(log N) : sorting
    - total : O(N * log N)
- 구현

```cpp
template<class ItemType>
void HeapSort(ItemType values[], int numValues){
	int index;
	//중간부터 시작해서 ReheapDown을 수행해 큰 값을 위로 올린다
	//중간부터 시작하는 이유는, 중간 인덱스보다 큰 인덱스에는 자식 노드가 없는 
	//leaf 노드이고, 이들은 이미 heap 조건을 만족하기 때문이다.
	for(index = numValues /2 - 1; index >=0; i--)
		ReheapDown(values, index, numValues - 1);

	for(index = numValues - 1; index >= 1; index--){ //아래 코드에 index - 1이 있으므로
		Swap(values[0], values[index]);
		ReheapDown(values, 0, index - 1); //이미 정렬된 하단 부분을 하나씩 제외
	}
}

template<class ItemType>
void ReheapDown(ItemType values[], int root, int bottom){
	int maxChild;
	int rightChild;
	int leftChild;

	leftChild = root * 2 + 1;
	rightChild = root * 2 + 2;

	if(leftChild <= bottom){
		if(leftChild == bottom)
			maxChild = leftChild;
		else{
			if(values[leftChild] <= values[rightChild])
				maxChild = rightChild;
			else
				maxChild = leftChild;
			if(values[root] < values[maxChild]){
				Swap(values[root], values[maxChild]);
				ReheapDown(values, maxChild, bottom);
			}
		}
	}
}

	
		
```
