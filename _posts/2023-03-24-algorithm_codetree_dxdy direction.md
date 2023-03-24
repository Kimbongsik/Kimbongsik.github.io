---
title: "[코드트리] 문자에 따른 명령2 (dx, dy 방향)"
last_modified_at: 2023-03-24T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - algorithm

tags:
  - 코드 트리
  - algorithm
---

dxdy 문제 기초 중의 기초..

**문제:**

좌표평면 위 (0, 0)에서 북쪽을 향한 상태에서 움직이는 것을 시작하려 합니다. N개의 명령에 따라 총 N번 움직이며, 명령 L이 주어지면 왼쪽으로 90도 방향 전환을, 명령 R이 주어지면 오른쪽으로 90도 방향전환을 하고, 명령 F가 주어지면 바라보고 있는 방향으로 한칸 이동하려고 합니다. 이동 이후 최종 위치를 출력하는 프로그램을 작성해보세요.

**입력 형식:**

첫 번째 줄에 문자 ‘L', ‘R', 그리고 'F’로만 이루어진 문자열이 하나 주어집니다.

- 1 ≤ 명령의 길이 ≤ 100,000

**출력 형식:**

최종 위치 (x, y)를 공백을 사이에 두고 출력합니다.

**풀이:**

```cpp
#include <iostream>
#include <string>
using namespace std;

int n;
int curr_dir = 3;
int x = 0;
int y = 0;
string dirString;
char dirChar;

//동 남 서 북
int dx[4] = {1,0,-1,0};
int dy[4] = {0,-1,0,1};

int main()
{
    cin >> dirString;
    for(int i = 0; i < dirString.size(); i++)
    {
        dirChar = dirString[i];
        
        if(dirChar == 'L')
            curr_dir = (curr_dir - 1 + 4) % 4;
            //3 -> 2, 2-> 1, 1-> 0, 0 -> 3

        else if(dirChar == 'R') 
            curr_dir = (curr_dir + 1) % 4;

        else
        {
            x += dx[curr_dir];
            y += dy[curr_dir];
        }

    }

    cout << x << " " << y;

		return 0;
}
```

**노트:**

연속된 n개의 Char을 계속 입력 받을 때, string으로 입력 받아 반복문에서 size() 함수를 통해 문자열의 길이만큼 반복하고, 인덱스를 통해 Char을 추출할 것.