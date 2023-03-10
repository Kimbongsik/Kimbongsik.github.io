---
title: "[컴퓨터구조] 블록 장치 & 플래시 메모리 & SSD"
last_modified_at: 2023-03-06T16:20:02-05:00

categories:
  - cs

tags:
  - cs
---

<br/>

  메모리를 읽거나 쓰려면 시간이 걸리는데, 대량 저장장치로 알려진 ‘디스크 드라이브’는 많은 데이터를 저장하기 좋은 장치다. 디스크 드라이브는 플래터에 비트를 저장한다. 디스크 헤드가 플래터에서 데이터를 읽는다. 디스크 드라이브는 다른 유형의 메모리에 비해 상대적으로 느린데, 방금 헤드를 지나간 데이터가 필요한 경우 그 데이터를 읽기 위해 해전판이 거의 한 바퀴 돌 때까지 기다려야 한다. 또 기계 부품이 시간이 지나면 낡기 때문에 베어링의 마모는 디스크 오류를 일으킨다. 디스크 드라이브는 디스크의 영역을 자화 시켜서 데이터를 저장하는데, 디스크 드라이브는 코어 메모리와 마찬가지로 비휘발성이다. 

 디스크 드라이브는 기록 밀도와 속도를 맞바꾼 기억 장치다. 속도가 느리지만 DRAM 등과 달리 주소나 데이터 연결을 위한 공간이 필요 없다. 디스크는 블록 단위로 주소를 지정해 읽는다. 블록은 역사적으로 섹터라고 불려왔고, 디스크에서 읽고 쓰기가 가능한 가장 작은 단위다. 디스크는 역사적으로 섹터당 512바이트를 저장했으나 최근에는 4096바이트를 저장한다. 이 말은, 디스크에서 한 바이트만 바꾸고 싶으면 전체 블록을 읽고 원하는 바이트를 바꾼 다음 전체 블록을 다시 써야 한다는 뜻이다. 

 모든 섹터에는 같은 수의 비트가 들어 있기 때문에 비트 밀도는 원판의 바깥쪽보다 안쪽이 더 높다. 바깥쪽 트랙에는 비트를 더 집어넣을 수 있는 여유가 많기 때문에 이런 방식은 낭비가 심하다. 최신 디스크들은 이 문제를 방사상 영역으로 구분해 해결한다. 이 방식에서는 내부 영역보다 외부 영역에 더 많은 섹터가 들어간다. 

- 디스크 드라이브 성능 표현을 위한 수치  

<br/>

1. 탐색 시간(seek time)

 암에는 헤드가 달려 있는데, 디스크는 헤드의 위치에 따라 트랙으로 나뉜다. 탐색 시간은 헤드를 한 트랙에서 다른 트랙으로 옮길 때 걸리는 시간이다.

2. 회전 지연 시간(rotational latency)

 원하는 데이터가 헤드 아래로 올 때까지 디스크가 돌아야 하는데, 이 때 걸리는 시간을 회전 지연시간이라고 한다. 보통 회전 지연 시간은 수 밀리초 수준이다.

플래시 메모리(flash memory)는 가장 최근에 나타난 EEPROM 유형의 매체이다. 음악 플레이어나 디지털 카메라 등의 응용에는 플래시 메모리가 적합하다.
플래시 메모리의 버킷은 DRAM보다 더 크고 잘 만들어져 있어 전자가 새지 않고, EEPROM보다 더 빨리 지우고 더 저렴하다. 또한 RAM처럼 원하는 위치를 마음대로 읽을 수 있다.
그러나 빈 플래시 메모리에 데이터를 기록하기 위해서는 먼저 0을 채워 넣어야 한다. 플래시 메모리의 0을 1로 바꿀 수는 있지만 전체를 지우지 않고 원하는 비트만 0으로 되돌릴 수는 없다. 모든 메모리를 다 지우는 것은 낭비가 심하므로 플래시 메모리의 내부는 블록으로 나눠서 블록 단위로 지우고 값을 쓸 수 있다. 따라서 플래시 메모리는 읽을 때는 임의 접근 장치이고 쓸 때는 블록 접근 장치이다. 디스크 드라이브는 점점 고체 상태 드라이브, *SSD(solid-state drive*)로 바뀌고 있다. SSD는 디스크 드라이브 모양의 패키지에 넣은 플래시 메모리와 같다. 플래시 메모리는 점점 낡기 때문에 SSD에는 여러 블록의 쓴 횟수를 기억해서 모든 블록이 가능하면 똑같은 수준으로 낡도록 조정하는 프로세서가 들어있다.
