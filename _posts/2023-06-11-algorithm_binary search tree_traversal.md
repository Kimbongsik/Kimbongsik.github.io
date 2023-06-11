---
title: "[자료구조/C++] 이진탐색트리, 트리 탐색(Tree Traversal)"
last_modified_at: 2023-06-11T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - algorithm
---
저번 시간에 이어서 포스팅을 해보록 하겠다.

트리를 출력하는 함수와 복사 생성자를 만들 것이다.

모두 재귀로 구현한다. 

# 구현

### PrintTree()

```cpp
void PrintTree(TreeNode* tree, std::ofstream& outFile){
	if(tree != NULL){
		PrintTree(tree->left, outFile); //트리의 가장 왼쪽으로 갔다가
		outFile << tree->info; //올라오면서 출력
		PrintTree(tree->right, outFile);
	}
}

template<class ItemType>
void TreeType::Print(std::ofstream& outFile) const{  
	PrintTree(root, outFile);
}
```

### Constructor & Deconstructor

```cpp
template<class ItemType>
TreeType<ItemType>::TreeType(){
	root = NULL;
}

void Destroy(TreeNode*& tree);

TreeType::~TreeType(){
	Destroy(root);
}

void Destroy(TreeNode*& tree){
	if(tree != NULL){
		Destroy(tree->left);
		Destroy(tree->right);
		delete tree;
	}
}
```

### CopyTree()

```cpp
TreeType::TreeType(cost TreeType& originalTree){
	CopyTree(root, originalTree.root);
}

void CopyTree(TreeNode*& copy, const TreeNode* originalTree){
	if(originalTree == NULL)
		copy == NULL;
	else{
		copy = new TreeNode; //복사하는 트리의 root node
		copy->info = originalTree->info;
		CopyTree(copy->left, originalTree->left);
		CopyTree(copy->right, originalTree->right);
	}
}
```

# 트리 순회(Tree Traversal)

이진 트리를 순회하는 방식에는 크게 3가지가 있다.

이때 순회란, 트리를 구성하는 모든 요소를 한 번씩 방문하는 것이다. 자주 써먹을 수 있는 방식이니 익혀두는 게 좋다. 

- **Inorder Traversal(중위 순회)**
    - 순서
        - left → root → right
- **Postorder Traversal(후위 순회)**
    - 순서
        - left → right → root
- **Preorder Traversal(전위 순회)**
    - 순서
        - root → left → right

중위, 후위, 전위라는 말은 ‘root’의 순서를 중심으로 보면 된다. 우리는 트리를 재귀로 구현하기 때문에 해당 root는 부모노드, 즉 ‘현재 해당하는 노드’이기 때문에, 이 부모노드를 중간에 방문하면 중위 순회, 맨 마지막에 방문하면 후위 순회, 맨 처음에 방문하면 전위순회가 된다. (쉽죠?)

그렇기 때문에, 사실 순회는 이게 전부다. 위에 Print문에서는 left → root(tree→info) → right 순으로 출력했으므로, 중위순회를 사용해 트리를 출력한 것이라고 보면 된다. (지난 번에 DeleteItem 구현 순서는, 후위 순회로 지웠다. 전위 순회로 지우게 되면 garbage가 생기기 때문.)

즉 위 Print 함수 구문을 순회명으로 바꾸어 구현만 해주면 된다. 

## Inorder (중위 순회)

순회하면서 노드를 삭제하든, 프린트를 하든 상관이 없다.

해당 예제에서는 트리 전체의 노드들을 큐에 담아보는 예제다.

```cpp
void InOrder(TreeNode* tree, QueType& inQue){
	if(tree != NULL){
		InOrder(tree->left, inQue);
		inQue.Enqueue(tree->info);
		InOrder(tree->right, inQue);
	}
}
```

## PostOrder (후위 순회)

```cpp
void PostOrder(TreeNode* tree, QueType& inQue){
	if(tree != NULL){
		PostOrder(tree->left, inQue);
		PostORder(tree->right, inQue);
		inQue.Enqueue(tree->info);
	}
}
```

## PreOrder(전위 순회)

```cpp
void PreOrder(TreeNode* tree, QueType& inQue){
	if(tree != NULL){
		inQue.Enqueue(tree->info);
		PreOrder(tree->left, inQue);
		PreOrder(tree->right, inQue);
	}
}
```

이제 아래와 같이 그려진 트리를 중위, 전위, 후위 순회 순서대로 작성해보자.

![tree](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/7b7bf839-5d40-4825-8092-8495dc21984a)


- 중위 순회: B→F→G→H→P→R→S→T→W→Y→Z
- 전위 순회: P→F→B→H→G→S→R→Y→T→W→Z
- 후위 순회: B→G→H→F→R→W→T→Z→Y→S→P

## GetNextItem & ResetTree

stack이나 queue, linked list 등의 자료구조에서 자주 구현해줬던 메소드가 바로 ‘GetNextItem’이다. 다음 순서에 올 아이템을 가져오는 기능인데, 이를 위해서 위의 순회를 이용해 큐에 아이템들을 저장한 후 차례대로 꺼내주는 방식으로 구현하면 된다!

ResetTree에서는 순회 함수들을 호출하여 큐에 저장하는 역할을 수행해주는 방식으로 구현한다.

```cpp
enum OrderType (PRE_ORDER, IN_ORDER, POST_ORDER};

class TreeType{
	public:
		//same as before
	private:
		TreeNode* root;
		QueType preQue; //전위, 중위, 후위 별로 큐를 미리 만들어준다.
		QueType inQue;
		QueType postQue;
	}
}

void TreeType::ResetTree(OrderType order){
	switch(order){
	case PRE_ORDER: PreOrder(root, preQue);
									break;
	case IN_ORDER: InOrder(root, inQue);
									break;
	case POST_ORDER: PostPrder(root, postQue);
									break;
	}
}

void TreeType::GetNextItem(ItemType& item, OrderTye order, bool& finished){
	finished = false;
	switch(order){
	case PRE_ORDER: preQue.Dequeue(item);
									if(preQue.IsEmpty())
										finished = true;
									break;
	case IN_ORDER: inQue.Dequeue(item);
									if(inQue.IsEmpty())
										finished = true;
									break;
	case POST_ORDER: postQue.Dequeue(item);
									if(PostQue.IsEmpty())
										finished = true;
									break;
	}
}

```