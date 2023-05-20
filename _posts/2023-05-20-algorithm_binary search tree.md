---
title: "[자료구조/C++] 이진탐색트리(Binary Search Tree) 재귀로 구현하기"
last_modified_at: 2023-05-20T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---

# 트리(Tree)
## 정의
* 노드의 연결로 구성되어 있다
* 사이클이 생기면 안 된다, 즉 노드의 연결을 따라갔을 때 다시 돌아올 수 있어선 안 된다

## 구성 및 정의
* Leaf Nodes: Child가 하나도 없는 노드. 트리의 가장 밑에 있는 노드들을 말함.
* Unleaf Nodes : Child가 하나 이상 있는 노드 
* Path: 경로. 거쳐가는 엣지(노드)들의 집합 
* Root Node: 깊이 Level 0에 위치한 최상위 노드 
* Parent Node: 부모 노드. 여러 개의 자식 노드를 가질 수 있음
* Child Node: 자식 노드. 단 하나의 부모 노드만을 가짐
* Descendant of a Node: 자손 노드. 해당 노드로부터 경로의 맨 마지막 노드까지에 있는 모든 노드들을 포함
* Ancestor of a Node: 조상 노드. 루트 노드부터 해당 노드까지 경로에 있는 모든 노드


# 이진 트리(Binary Tree)
## 정의
* 각 노드들이 모두 두 개 이하의 자식 노드를 가짐


## 완전 트리(Full Tree)
* 모든 리프 노드가 같은 계층(level)에 있음
* 모든 언리프 노드가 두 개의 자식 노드들을 가지고 있음
* 노드 개수 : N = 2^h - 1
* 높이 : h= log(N+1)—> O(logN) 

## 이진 트리 특징
* 각 노드들은 서로 다른 데이터들을 가지고 있음
* 이에 따라, ”greater than”, 또는 ”less than” 비교를 통해 노드를 정리할 수 있음
* 해당 노드의 **키보다 작은 노드는 왼쪽 subtree에, 큰 노드는 오른쪽 subtree에 위치함**
* 어떤 순서로 배치하느냐에 따라 트리의 모양이 달라진다
	* J E F T A 의 순서대로 배치하는 트리와, A E F J T 순서대로 배치하는 트리를 그려보자


## 탐색(Searching)
### 기존 트리의 경우
* 트리의 노드 수 가 N개면 N번 모두 비교해야 한다는 단점이 있다
* 최악의 경우 시간 복잡도는 O(N)

### 이진 트리의 경우 탐색 방법
1. 루트 노드에서 시작
2. 찾고자 하는 아이템과 노드의 아이템과 비교
3. 찾고자 하는 아이템이 더 작으면, left node와 비교
4. 찾고자 하는 아이템이 더 크면, right node와 비교
5. 반복

* 비교했을 때 동일하면 -> found == true
* 비교한 노드가 leaf 노드일 때 동일하지 않으면 -> found == false
* 시간 복잡도가 개선된다 O(log N)



## 구현

* 즐거운 구현 시간!

노드는 아래와 같이 구성해준다.
각 노드에는 아이템을 저장할 info와, 양쪽 자식 노드를 가리킬 포인터 변수를 가지고 있다.

```cpp
template<class ItemType>
struct TreeNode{
	ItemType info;
	TreeNode* left;
	TreeNode* right;
}
```

이제 트리 타입에 대해 정의해준다. ItemType을 선언해 준 이유는 노드에 저장할 아이템의 타입을 국한해주지 않기 위해서다. (char, int 등..)

```cpp

#include <fstream.h>

template<class ItemType>
struct TreeNode;

enum OrderType {PRE_ORDER, IN_ORDER, POST_ORDER}

template<class ItemType>
class TreeType{
public:
	TreeType();
	~TreeType();
	TreeType(const TreeType<Itemtype>&);
	void operator=(const TreeType<ItemType>&); //트리 복사를 위한 연산자 오버로딩

	void MakeEmpty();
	bool IsEmpty() const;
	bool IsFull() const;
	int LengthIs() const;
	void RetrieveItem(ItemType&, bool& found);
	void InsertItem(ItemType);
	void DeleteItem(ItemType);
	void ResetTree(OrderType);
	void GetNextItem(ItemType&, OrderType, bool&);
	void PrintTree(ofstream&) const;

private:
TreeNode<ItemType>* root;
};
		

```

이제 내용을 구성해보자.

### IsFull() & IsEmpty()

```cpp

bool TreeType::IsFull() const{
	NodeType* location;
	try{
		location = new NodeType; // 새로운 노드 생성이 가능하면
		delete location;
		return false; //노드가 다 차지 않았음
	}
	catch(std::bad_alloc exception){ //그렇지 않으면 bad alloc exception
		return true;
	}
}

bool TreeType::IsEmpty() const{
	return root == NULL; //루트가 Null인가 아닌가?
}

```

### LengthIs()
잠깐, LengthIs() 함수를 구현하기 전에 만들어야 할 것이 있다.
노드의 개수를 세야 한다. 
모든 노드는 left, right를 가지고 있으므로 노드를 셀 때, Recursive 하게 구현이 가능하다. 재귀로 구현하기 위해 base case와 general case를 따져보자.

* base case : 트리가 비었을 때
* general case : CountNodes(Left(tree)) + CountNodes(Right(tree)) + 1

자, 그럼 다시 구현이다.
CountNodes 메소드를 재귀적으로 구현하고, 해당 함수를 호출해 줄 함수로 LengthIs를 만들어준다. 

```cpp

int TreeType::LengthIs() const{
	return CountNodes(root);
}

int CountNodes(TreeNode* tree){
	if(tree == NULL)
		return 0;
	else
		return CountNodes(tree->left) + CountNodes(tree->right) + 1;
}

```


### RetrieveItem()

```cpp
void TreeType::RetrieveItem(ItemType& item, bool& found){
	Retrieve(root, item, found);
}

void Retrieve(TreeNode* tree, ItemType& item, bool& found){
	if(tree == NULL){ //base case 1
		found = false;
	}
	else if(item < tree->info){
		Retrieve(tree->left, item, found); //왼쪽 트리로 내려가기
	}
	else if(item > tree->info){
		Retrieve(tree->right, item, found); //오른쪽 트리로 내려가기
	}
	else{ //base case 2. 아이템을 찾음
		item = tree->info;
		found = true;
	}
}

```

### Insert()
```cpp
void Insert(TreeNode*& tree, ItemType item){
	if(tree == NULL){
		tree = new TreeNode;
		tree->right = NULL;
		tree->left = NULL;
		tree->info = item;
	}
	//좌,우로 내려가면서 삽입할 위치를 찾음
	//해당 방법은 트리의 중간에 삽입하는 게 아니라 순서대로, 즉 항상 리프노드로 삽입함
	else if(tree->info > item){ 
		Insert(tree->left, item);
	}
	else{
		Insert(tree->right, item);
	}
}
```

### DeleteItem()

노드를 삭제할 때는 삭제할 아이템을 찾고, 그 뒤에 삭제해야 한다. 중요한 것은 해당 노드를 삭제한 후에도 이진 트리 모양을 유지해야 한다는 것이다. 
Leaf 노드를 삭제할 경우 해당 노드의 연결만 끊어지면 되지만 중간에 있는 노드를 삭제하는 경우, 생각해야 될 게 좀 있다.

방법은 다음과 같다.

1. 전임자(Predecessor)를 찾는다
2. 전임자를 삭제할 노드의 위치에 넣고
3. 전임자를 삭제한다

즉, 삭제할 노드와 전임자를 바꿔치기하면 된다. 

* 전임자란? : 해당 노드 다음으로 큰 노드. 즉, 노드의 left subtree에서 가장 큰 노드를 말한다.
그렇다면 전임자를 찾기 위해선 해당 노드의 left subtree에서 가장 오른쪽에 위치한 leaf node를 찾으면 된다. 전임자는 자식 노드가 없고, 삭제할 노드 기준 left subtree 중 가장 아이템 값이 크며, 삭제할 노드보다는 값이 작기 때문에 left subtree와 rightsubtree를 이어줄 parent node가 될 수 있다. 

```cpp

//노드 삭제
void DeleteNode(TreeNode*& tree){
	ItemType data;
	TreeNode* tempPtr;
	tempPtr = tree; 
	//지우고자 하는 트리 노드의 좌 또는 우가 NULL이면 그냥 삭제해서 당겨주면 된다
	if(tree->left == NULL){ 
		tree = tree->right;
		delete tempPtr;
	}
	else if(tree->right == NULL){
		tree = tree->left;
		delete tempPtr;
	}
	//그렇지 않은 경우
	else{
		GetPredecessor(tree->left, data); //전위자를 찾아서
		tree->info = data; //트리의 info에 전위자의 info를 넣어주고
		Delete(tree->left, data); //전위자를 찾아서 지워준다
	}

}

//삭제할 노드 탐색
void Delete(TreeNode*& tree, ItemType item){
	if(item < tree->info) //삭제하고자 하는 아이템이 info보다 작으면
		Delete(tree->left, item); //왼쪽으로 타고 내려감
	else if (item > tree->info)
		Delete (tree->right, item); //오른쪽으로 타고 내려감
	else //base case
		DeleteNode(tree); //삭제할 노드를 찾았으므로, 삭제
}

//tree에는 반드시 삭제하고자 하는 노드의 left를 넣어줘야 한다.
void GetPredecessor(TreeNode* tree, ItemType& data){
	while(tree->right != NULL)
		tree = tree->right;
	data = tree->info;
}

```
