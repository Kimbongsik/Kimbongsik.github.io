---
title: "[C++] C++의 iterator"
last_modified_at: 2025-02-11T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - C_C++

tags:
  - C_C++
---

요즘 코딩 테스트를 C++로 준비하고 있는데, 나는 그동안 개발하면서 C++을 C처럼 쓰고 있었구나 싶었다. 사실 임베디드 시스템에서는 C++이 꽤 무거워 쓰지 않는 경우가 많아, C++의 다양한 툴들이 더욱 낯설게 다가온 것 같다. 

아무튼,  C++에서는 vector, list, map 등 다양한 컨테이너를 제공하고 있는데, 해당 컨테이너의 원소에 잘 접근하기 위해서는 이 ‘iterator’라는 걸 잘 알아둬야 할 것 같다.

# std::iterator란?

iterator는 C++에서 사용하는 일종의 포인터이다.

C++에서 제공하는 다양한 컨테이너의 원소에 동일한 방법으로 접근하기 위해 제공되는 객체이다. 따라서 Vector, List, String 등, C++에서 제공하는 다양한 컨테이너의 요소에 접근할 때 복잡하게 생각할 것 없이, 이 반복자(iterator)의 사용법에 대해서 잘 알아두면 활용하기 쉽다.

예를 들어보자.

> **Vector vs List**
> 

Vector는 배열 기반 컨테이너로, 각 요소들에 대해 연속된 메모리 공간을 할당한다. 따라서 **인덱스를 통해 요소에 접근이 가능하다.**

하지만 List는? C++에서 제공하는 list 는 노드 기반 컨테이너로, 이중 연결 리스트(doubly linked list)로 구현되어 있다. 이러한 연결 리스트는 요소들에 대해 연속된 메모리 공간을 사용하지 않는다. 즉, **인덱스를 통해 요소에 접근할 수 없다.**

> **결론**
> 

C++에서는 다양한 컨테이너와 도구를 제공한다. 위에서는 vector와 list만을 예로 들었지만, map, set, string 등, 정말 다양한 형태의 컨테이너들이 존재한다. 이 컨테이너들마다 각기 다른 요소 접근 방법 가지고 있고, 이들을 모두 외워야 한다면 개발 편의성이 정말 낮아질 것이다. 그래서 사용하는 게 바로 iterator다.

# 사용 방법

그럼 iterator의 사용 패턴을 외워둔다면,  다양한 컨테이너의 원소에 접근할 때 아주 요긴하게 쓰일 것이다. iterator는 여러 C++ STL과 같이 잘 쓰이기 때문이다.

## begin(), ::end()

- begin() : 첫 번째 원소 위치 반환
- end(): 마지막 원소 다음 위치 반환

```cpp
vector<int> v;

v.push_back(1);
v.push_back(2);
v.push_back(3);

vector<int>::iterator iterator_begin = v.begin();
vector<int>::iterator iterator_end = v.end();

for(int i = 0; iterator_begin != iterator_end; iter_begin++)
	cout << *iterator_begin << endl;
```

위는 벡터로 접근한 예시이다. 하지만 iterator는 다른 컨테이너에서도 자주 사용한다고 했다. 즉, list에서도 같은 접근이 가능하다.

```cpp
list<int> l;
l.push_back(1);
l.push_back(2);
l.push_back(3);

list<int>::iterator_begin = l.begin();
list<int>::iterator_end = l.end();

forfor(int i = 0; iterator_begin != iterator_end; iter_begin++)
	cout << *iterator_begin << endl;
```

## rbegin() & rend()

- rbegin() : 마지막 원소 위치 반환
- rend(): 첫 번째 원소의 이전 주소 반환

```cpp
vector<int> v = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
for (auto iter = v.rbegin(); iter != v.rend(); iter++)
    std::cout << *iter << " "; // 10 9 8 7 6 5 4 3 2 1
```

iterator 선언이 좀 길고 복잡하기 때문에, 위와 같이 반환값을 ***auto***로 해주면 편하다.

auto 는 선언의 초기화 식에서 형식이 추론되는 변수를 선언한다. 즉, 복잡한 형식의 변수를 선언하는 것을 피하기 위해 사용하는 변수 선언 방식으로, 알맞게 사용하면 아주 편하다.

## <algorithm> find()

```cpp
std::find(begin, end, value)
```

- array, vector 등에서 원하는 값을 탐색하는 함수
- 범위 내 원소 중, value와 일치하는 첫 번째 원소의 iterator를 리턴
- 찾고자 하는 값이 없으면 end를 리턴

```cpp
vector<int> v = {1,2,3,4,5,6,7,8,9,10};
auto it = std::find(v.begin(), v.end(), 5);

if(it != v.end())
	cout << *it << " : " << std::distance(v.begin(), it) <<"\n";
else
	cout << "not found\n";
```

## <algorithm> remove()

```cpp
std::remove(begin, end, value)
```

- 컨테이너의 특정 원소를 제거하기 위해 사용하는 함수
- 제거 후 컨테이너의 원소 개수는 변하지 않음. 제거할 원소 위치 뒤에 있는 원소들을 앞으로 복사하여 덮어 씀
- 요소가 제거된 이후의 iterator를 리턴(마지막 요소 바로 다음)

```cpp
vector<int> v = {1,2,3,4,5,6,7,8,9,10};
auto rem_v = remove(v.begin(), v.end(), 4);
for(auto it = v.begin(); it != rem_v; ++v) cout << *it << ' ';
//1 2 3 5 6 7 8 9 10
```

## iterator erase()

```cpp
erase(const_iterator position) // 지운 요소 뒤의 위치를 리턴
```

- 컨테이너의 특정 원소를 제거하기 위해 사용하는 함수
- remove()와 달리, 컨테이너의 capacity(용량)가 줄어듦
- 지운 요소 뒤의 위치를 리턴

```cpp
vector<int> v = {1,2,3,4,5,6,7,8,9,10}; 
v.erase(v.begin() + 4); // 1 2 3 4 6 7 8 9 10
```

## iterator insert()

```cpp
insert(const_iterator position, const value_type& value);
```

- position에 value를 삽입

```cpp
vector<int> v = {1,2,4};
auto it = v.insert(v.begin() + 2, 3};

for(int i : v) cout << i <<" "; //1 2 3 4 
```

# 주의 사항(iterator 종류)

좀 헷갈리는 건, C++에서 사용하는 이터레이터가 여러 종류가 있다는 것이다. 위에 열거한 몇 메소드를 보면 알겠지만, 어떤 건 <algorithm> 헤더파일에서 사용하고, 어떤 건 STL 컨테이너에서 사용한다.

그러니 주의해서 사용해야 한다.

## <algorithm> 헤더에서 사용하는 iterator

- 비멤버 함수 기반의 iterator
- **컨테이너에 종속되지 않음(범용적으로 사용)**
- std::find, std::sort, std::remove 등
- **컨테이너의 이터레이터를 받아서 동작함**
    - ex) find(container.begin(), container.end(), value)

## STL 컨테이너에서 제공하는 iterator

- C++ STL(std::vector, std::list, std::map 등)에는 각 **컨테이너 전용 이터레이터**가 내장되어 있음
- **멤버 함수 기반의 iterator** ( ex: container.begin(), container.end())
- 각 컨테이너마다 최적화된 iterator를 제공함
    - std::vector<int> iterator는 std::vector 전용으로 최적화 된 iterator