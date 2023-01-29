---
title: "[백준] 1655번: 가운데를 말해요"
last_modified_at: 2023-01-29T16:20:02-05:00
categories:
  - algorithm

tags:
  - 백준
---


### 필요 알고리즘:

우선순위 큐, 최대힙, 최소힙



### 문제:

백준이는 동생에게 "가운데를 말해요" 게임을 가르쳐주고 있다. 백준이가 정수를 하나씩 외칠때마다 동생은 지금까지 백준이가 말한 수 중에서 중간값을 말해야 한다. 만약, 그동안 백준이가 외친 수의 개수가 짝수개라면 중간에 있는 두 수 중에서 작은 수를 말해야 한다.

예를 들어 백준이가 동생에게 1, 5, 2, 10, -99, 7, 5를 순서대로 외쳤다고 하면, 동생은 1, 1, 2, 2, 2, 2, 5를 차례대로 말해야 한다. 백준이가 외치는 수가 주어졌을 때, 동생이 말해야 하는 수를 구하는 프로그램을 작성하시오.


### 입력:

첫째 줄에는 백준이가 외치는 정수의 개수 N이 주어진다. N은 1보다 크거나 같고, 100,000보다 작거나 같은 자연수이다. 그 다음 N줄에 걸쳐서 백준이가 외치는 정수가 차례대로 주어진다. 정수는 -10,000보다 크거나 같고, 10,000보다 작거나 같다.



### 코드(오답):

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

static int arr[100000] = { 0, };
int answer[100000] = {0, };

int main(){
	cin.sync_with_stdio(0);
  cin.tie(0);

	int n;
	int count = 0;
	cin >> n;
	
	for (int i = 0; i < n; i++){

		cin >> arr[i];
		count++;

		if (count % 2 == 0 && i > 0){

			if (arr[(count/2)-1] > arr[count/2])
				answer[i] = arr[count / 2];

			else
				answer[i] = arr[(count / 2) - 1];
						
		}

		else if (count % 2 == 1){

			sort(arr, arr + i+1);
		
			answer[i] = arr[count/2];
			
		}

	}

	for (int j = 0; j < n; j++)
		cout << answer[j] << "\n";

}
```



→ 다음과 같이 코드를 작성했으나 시간 초과 오류가 떴음. (어쩐지 쉽다 했다..)

→ n의 최대 입력이 10만개이므로, 정수 입력마다 정렬하게 되면 시간 초과가 발생.

→ 효율적으로 중간값을 구할 필요가 있음.



### 알고리즘

→ 최대 힙과 최소 힙

→ 최대 힙의 루트 노드가 중간값이 되게 만들어야 함



### 코드(정답)

```cpp
#include <iostream>
#include <queue>
using namespace std;

int N;
int num[100000];
int answer[100000];
static int cnt = 1;

void swap(priority_queue<int, vector<int>, greater<int>>& min_heap, priority_queue<int, vector<int>, less<int>>& max_heap)
{
	if (max_heap.top() > min_heap.top()) {
		int min_top = min_heap.top();
		int max_top = max_heap.top();
		max_heap.pop();
		min_heap.pop();
		max_heap.push(min_top);
		min_heap.push(max_top);
	}
}

int main()
{
	cin >> N;

	priority_queue<int, vector<int>, greater<int>> min_heap;
	priority_queue<int, vector<int>, less<int>> max_heap;

	for (int i = 0; i < N; i++)
	{
		cin >> num[i];
		if (i == 0) {
			max_heap.push(num[i]);
			answer[0] = max_heap.top();
			cnt++;
		}
		
		else if (cnt % 2 == 0) {
			min_heap.push(num[i]);
			swap(min_heap, max_heap);
			answer[i] = max_heap.top();
			cnt++;
		}

		else if (cnt % 2 == 1) {
			max_heap.push(num[i]);
			swap(min_heap, max_heap);
			answer[i] = max_heap.top();
			cnt++;
		}
	
	}

	for (int j = 0; j < N; j++)
		cout << answer[j] << '\n';
}
```
