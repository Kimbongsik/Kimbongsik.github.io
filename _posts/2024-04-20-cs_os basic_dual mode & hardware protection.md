---
title: "[운영체제] [Basic] OS 이중모드 & 하드웨어 보호"
last_modified_at: 2024-04-20T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - cs

tags:
  - cs
---

## 이중모드

한 컴퓨터를 여러 사람이 동시에 사용하는 환경이나, 한 사람이 여러 개의 프로그램을 동시에 사용할 때 한 사람의 고의나 실수가 프로그램 전체에 영향을 줄 수 있다.
이중모드는 말 그대로 모드를 두 개를 둠으로써, 이 같은 상황에서 발생하는 문제를 방지한다.

* 관리자 모드(supervisor mode = monitor mode = privileged mode)
* 사용자 모드(user mode)

관리자 모드 즉, 특권 모드에서만 내릴 수 있는 명령을 특권 명령(privileged instruction)이라 하는데, STOP, HALT, REST, SET_TIMER 등이 있다.  
따라서 유저 모드에서는 특권 명령을 사용할 수 없다.  

CPU 내부에는 비트들의 모음인 register가 있는데, 이중 모드는 이 레지스터의 한 비트를 Monitor Bit로 할당하여 나타낸다. 
Monitor Bit가 1이면 시스템 모드로, 0이면 유저 모드로 사용하는 방식이다. 이러한 플래그(flag)에는 Carry, Negative, Zero, Overflow 등이 있다.

컴퓨터를 부팅 시키고 한 프로그램을 가동 시키는 과정을 살펴보자.  

- 컴퓨터 부팅(하드 디스크에서 os가 메인 메모리로 올라감. Monitor bit =1) => 관리자 모드
- 푸로그램 실행 과정 => 관리자 모드
- 프로그램 실행 중(CPU가 프로그램의 레지스터를 읽어와 실행 시, OS는 Monitor bit를 0으로 만듦) => 유저 모드

정리하면,

- 운영체제 서비스 실행될 때는 관리자 모드
- 사용자 프로그램 실행될 때는 사용자 모드
- 하드웨어/소프트웨어 인터럽트 발생 시 관리자 모드
- 운영체제 서비스 종료시 다시 사용자 모드

프로그램에서 정보를 저장하기 위해선 어떻게 해야 할까? 예를 들어 한 게임에서 지금까지의 스코어를 하드 디스크에 저장하고 싶을 시, 프로그램이 직접 하드 디스크로 접근하지 않고 os가 대신 저장해준다.  
게임과 같이 여타 유저 프로그램이 하드 디스크에 직접 접근하게 되는 것은 하드 디스크에 있는 다른 사람의 파일 또한 접근할 수 있다는 뜻으로 보안에 심각한 문제를 일으킨다.
게임 프로그램은 os에게 정보 저장을 위한 요청을 하고, software interrupt를 사용하여 IRS로 넘어간다. 이 때 OS는 관리자 모드로 바뀌며, 스코어 저장이 끝날 시 다시 사용자 모드로 돌아간다. 

일반적인 프로그램 실행은 다음과 같다.  

-user mode > 키보드, 마우스 > system mode(ISR) > user mode >모니터, 디스크,프린터 > system mode > user mode  

대부분의 CPU는 이런 이중모드를 지원한다. 일반 유저가 특권 명령을 사용하려 할 시, CPU가 monitor bit가 0인 것을 확인 하고 내부 인터럽트가 발생했다고 판단해 프로그램을 강제 종료시킨다. 
이렇듯 이중모드는 보호(protection)와 관련이 있다. 

## 하드웨어 보호

### 1. 입출력 장치 보호(I/O protection)

문제점>

* 여러 개의 입출력 장치 사용의 혼선
* 사용자가 다른 사용자의 데이터의 입출력을 함
* 여러 사용자의 데이터가 섞여 출력

해결>

입출력 명령을 특권 명령으로 만든다. (IN, OUT)

어떤 프로그램이 프린트를 사용하려고 할 때, 프로그램은 user mode이므로 프린트를 가동 시키는 OUT 명령을 내를 수 없다. 실행 시 내부 인터럽트를 발생시켜 ISR로 넘어가 프로그램을 강제 종료 시킨다. 
사용을 위해, INT 명령을 내려(소프트웨어 인터럽트) OS가 프린터를 가동시킨다. 그러나 디스크 내에 있는 아무 파일을 출력할 수 없기 때문에, 이 출력 요청이 올바른지 확인하는 루틴 또한 OS에 포함되어 있다. 
올바른 요청이 아닐 시 해당 파일을 읽을 수 없게 되어 접근하면 안 되는 다른 사람의 파일은 읽을 수 없게 된다.

### 2. 메모리 보호(Memory protection)

문제점>

* 메인 메모리 안에는 여러 user program들이 있는데, 자신의 메모리 영역만 보는 게 아니라 다른 user program이나 os를 건드는 일이 발생할 수 있다. 
* 다른 사용자의 메모리 또는 os 영역 메모리에 접근

해결> 
1. CPU는 메인 메모리의 몇 번째 내용을 읽겠다는 Address bus를 보내고 해당 내용이 담긴 Data bus를 받는다. 일반 유저 프로그램이 다른 영역에 침범하지 않게 하기 위해서는 이 Address bus를 차단하면 된다.
그런데 그렇게 되면 자기 영역에도 접근할 수 없게 된다. -> X

2. Address bus에 문지기를 세운다. 이를 MMU(Memory Management Unit)이라 하는데, MMU 설정은 특권명령을 통해서만 가능하다. MMU는 address bus 중간에 설치된 하드웨어 칩으로, 두 개의 레지스터를 통해 프로그램의 주소 범위를 저장한다.
User1의 주소가 1024번지 - 4048번지 사이이라고 가정하자. User1이 수행되는 동안 os는 해당 프로그램의 주소 범위를 MMU에 설정한다. 이 때 base register는 1024, limit register는 4048이 된다. CPU가 주소를 낼 때 범위가 해당 번지 내에 있을 때 address bus를 통과 시키고, 그렇지 않으면 MMU에서 내부 인터럽트를 발생시켜 CPU가 ISR로 이동해 프로그램을 강제 종료 시킨다. 

-> 잘못된 메모리 접근, 즉 다른 사용자 또는 운영체제 영역 메모리에 접근하려는 시도  
:Segment voilation

### 3. CPU 보호(CPU protection)

문제점>

* 고의, 혹은 실수로 인한 CPU 독점. 예를 들어, while(n=1) ... 이라는 무한 루프.

해결>
Timer를 두어 일정 시간이 경과할 시 타이머 인터럽트를 건다. 인터럽트 발생 시 운영체제 내의 ISR로 이동하여 해당 ISR에서 프로그램의 CPU 점유 시간을 측정하고 적절한 분배가 되도록 조정한다.  
한 프로그램의 CPU 점유 시간이 비정상적으로 오래 걸릴 시에 다른 프로그램으로 CPU를 전환시켜준다.  