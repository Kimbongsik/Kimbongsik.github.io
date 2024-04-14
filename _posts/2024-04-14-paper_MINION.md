---
title: "Securing Real-Time Microcontroller Systems through Customized Memory View Switching"
last_modified_at: 2024-04-14T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - papers

tags:
  - NDSS
  - embedded
  - security
  - memory protection
---

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/61307baa-fc44-4383-95d1-fb3304e54838)

# Abstract

- MINION을 제안
    - Real-time microcontroller에서, 메모리 공간에 대한 가상 파티션을 생성하고 메모리 접근 제한을 컨트롤 하게 함
    - Offline static analysis를 통해 real time process가 접근 가능한 메모리 영역을 자동으로 식별
    - 하드웨어 기반 제한으로 런타임 메모리 접근 컨트롤 가능
- 결과
    - 프로세스가 접근할 수 있는 메모리 영역을 줄임으로써, 다양한 공격으로부터 효과적으로 메모리를 보호
    - 보통은 메모리 분리 기법이 퍼포먼스 오버헤드를 높이지만, MINION은 그렇지 않고 real-time 특성을 유지하면서 가볍게 설계됨

# Introduction

- Process Memory Isolation
    - 각 프로세스의 가상 메모리 공간을 다른 프로세스로부터 Isolation
    - 제한된 메모리 접근으로 인해 공격자가 victim process로 프로세스를 공격하기 쉽지 않음
- Kernel Memory Isolation
    - 커널 메모리 공간을 user process context로부터 권한을 분리하여 Privilege escalation attack을 커널에서 수행하기 어렵게 함. (예를 들어, illegally한 커널 실행 context를 user execution context로부터 얻는 것)

## Problems

- 67개의 real time MCS 조사 결과 → kernel&process isolation 구현한 건 아무 것도 없었음
    
    ![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/8e4b2b0c-0ced-47e1-8ce1-8690820b7b50)
    
- Virtual memory를 통한 Process memory isolation이 지원되지 않는 건, MMU를 사용하지 않기 때문임
- Kernel memory isolation이 지원되지 않는 건, privilege mode switching으로 인해 발생하는 오버헤드가 real-time성을 해치기 때문임
    - 실제로 MCS에서 동작하는 몇몇 RTOS에서 옵션으로 Kernel memory isolation을 지원하기도 하나, 대부분 real-time성이 방해 받아 off 된 상태로 실행됨
- Memory Isolation이 지원되지 않으면 발생하는 문제
    - 공격 표면이 커짐
    - virtual memory가 없으면 system layout이 single memory space가 되며, 모든 소프트웨어 및 OS가 같은 권한 모드에서 실행될 뿐 아니라 전체 shared memory space에 접근할 수 있음
        
        ![3](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/fc14e291-3fad-4418-bf85-cdc36329e881)
        
- 기존 연구
    - 임베디드 디바이스에서 Isolation을 수행하려는 여러 시도가 있었으나, 퍼포먼스 적인 문제들이 있었음
    - 또한 어떤 subject를 어떤 메모리 영역에 접근 가능하게 할지에 대한 메뉴얼이 제공되어야 했음. 왜냐하면 메모리 바운더리를 정의하는 게 어려웠기 때문임
    - 최근 연구에서는 민감한 명령어를 자동으로 식별하고 실행되기 전에 하드웨어 기반 접근 제어를 위한 권한을 상승 시키는 방식을 제안함
    - 그러나 real-time성을 유지하기 위해, 모든 민감한 명령어에 대한 실행 권한을 상승시키는 것이 항상 허용되는 것은 아님. 이러한 명령어가 실시간 MCS에서 매우 자주 실행될 수 있기 때문
    - **즉, 기존 기술은 모든 주변 장치의 I/O 작업에 대한 권한 모드를 상승시켜야 하므로 실시간 요구사항을 충족하지 못한다는 문제점을 가지고 있음**

## Propose

- MINION : 자동으로 real-time MCS의 메모리 공간을 줄이면서 real-time성을 충족시키는 기술
- 핵심 기술 : Memory view switching mechanism for memory isolation
    - 주기적인 권한 변경을 피함
- Overview
    
    ![4](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/11a08d99-1ddb-44f1-8ee9-e56a24174c72)
    
    1. MINION은 각 프로세스를 동작시킬 수 있는 메모리 영역의 적절한 최솟값을 잡음. 실시간 MCS의 모든 메모리 유형(code, data, peripheral mapped memory)을 종합적으로 고려하는 point to 분석을 기반으로 보수적인 static analysis를 수행
    2. analysis 결과를 기반으로 pre-process memory view를 구성함. 실시간 MCS에서 현재 메모리 보호 하드웨어의 한계를 극복하기 위해 클러스터링 분석을 수행하여 기본 하드웨어 메커니즘과 호환되도록 메모리 뷰를 조정
    3. 그리고 view-switcher를 사용하여 memory isolation을 수행함. view switcher는 특권 모드에서 운용되는 유일한 TCB(Trusted Computing Base)이며, 비특권 모드에서 실행되는 RTOS 및 프로세스와도 분리되어 있음

- 기존 가상 메모리 기술 비교
    - 메모리 뷰를 이루는 메모리 블록은 연속적이지 않음 (각 프로세스의 메모리 블록이 monolithic memory space에서 섞여있으므로)
    - 프로세스 실행 및 권한 모드 실행(hardware interrupt 및 system call 호출로 인한)마다 주기적으로 실행되는 OS kernel과는 별개로 **view switcher는 프로세스 간 context switching이 발생할 때만 실행됨**
        - 즉, OS의 실행과 관계 없이 실행되는 view switcher는 아주 약간의 오버헤드만 발생함.
        - 이에 따라 real-time 충족 가능
            - 하드웨어 제한이 있는 경우 76%, 하드웨어 제한이 없는 경우 93%

# Background and motivation

## A. ARM Cortex-M and Cortex-R

- 둘 다 실시간성을 충족하는, Real-time system에서 가장 메이저하게 사용되는 MCS.
- fast interrupt handling, high responsiveness to hardware events(timer interrupt, peripheral event) 지원

## B. Memory Protection Unit

- 하드웨어적으로 memory isolation을 해주는 장치
- ARMv7 이후의 Cortex-M & Cortex-R 시리즈에 탑재되어 있음
- 그러나 MMU와 달리, 가상 메모리나 페이지 테이블 기반 메모리 접근 제어 기능을 제공하는 것은 아님
- 대신, MPU는 고정된 수의 하드웨어 레지스터를 제공하며, 각 레지스터는 물리 메모리에 대한 영역 별 액세스 제어 규칙을 지정함
- 메모리에 있는 페이지 테이블을 탐색하지 않아도 되므로, 빠르게 access control rule을 읽을 수 있음
- 각 메모리 영역마다 두 개의 permission set이 있음. 하나는 프로세서가 privileged mode일 때, 다른 하나는 unprivileged mode일 때 사용됨
- 각 permission set은 read / write / execution permission을 포함함
- 만약 접근 제어 룰에 위반하는, 권한 없는 메모리 접근이 발생하면 프로세서는 permission fault를 발생시킴 (MMU 기반 시스템의 page fault처럼)
- privileged / unprivileged mode에서의 메모리 접근은 access control rule이 분리된 permission set을 가지고 있으므로, 별개로 동작함
- MPU가 할당 가능한 영역은 8개-16개의 영역임

## C. Motivation of MINION

### Limitations of Current RTOSs

- RTOS에서 kernel memory isolation을 제공하기는 하나, Real-time성을 지키기 위해서 사용되지 않는 경우가 대부분임.
- I/O 호출이 빈번하게 발생하는데, 그때마다 kernel을 호출해줘야 하므로 권한 switch 시 발생하는 오버헤드가 크기 때문에 real-time성이 깨지게 됨
- UAV 시스템인 3DR IRIS+ Quradcopter에서 커널 메모리 격리를 수동으로 활성화하고 실시간 제약 조건을 충족하는지 관찰한 결과, 기본적으로 커널 메모리 격리를 활성화 시키지 않으며 강제로 활성화 할 경우 Real-time 조건을 충족하지 못하는 것을 확인함

### MINION Approach

- 모든 소프트웨어가 권한 모드에서 동작하는 기존 real time MCS와 달리, MINION은 모든 소프트웨어를 unprivileged mode에서 동작 시키며, 오직 view switcher만 privileged mode에서 동작함.
- 이에 따라 switching overhead를 감소시킴
- 메모리 뷰는 프로세스별로 적용되므로, view switcher는 MPU를 재구성하기 위해 context switching마다 한 번만 발생하면 됨
    - 메모리 뷰의 메모리 범위는 하드웨어에 종속적

# 3. Treat Model and Assumptions

- memory corruption 무력화를 목표로 함
    - MCS 펌웨어의 모든 보안 관련 패치 중 50%가 memory corruption vulnerability 해결과 관련되어 있었음
- MCS에 MPU가 장착되어 있고 하드웨어 기반 권한 분리를 지원한다고 가정함(Treat Model)

# 4. Design of MINION

## A. Firmware Analysis

- 실시간 프로세스가 런타임에 어떤 리소스에 액세스하는지 식별하기 위해 펌웨어 분석 실행
- 분석 수행 방법
    - LLVM IR 비트코드에서 수행
    - 보수적인 메모리 분석을 위해 정적 분석 수행
1. code section 도달 가능성 분석
    - 프로세스의 entry function에서 도달할 수 있는 모든 함수를 찾음
        - 프로세스의 시작 기능 & 하드웨어 이벤트 발생 시 트리거 되는 인터럽트 핸들러 포함
    - 전체 펌웨어의  CFG 구성
        - direct call & indirect call 포함
        - 런타임에 실행될 수 있는 함수를 표시
            - 인터럽트 핸들러, 프로세스 생성 담당 함수 포함
2. data section 접근성 분석
    1. Global Data의 경우
        - forward slicing을 통해 global data에 대한 memory access 식별
        - 어떤 함수가 전역 데이터에 직접 또는 간접적으로 접근하는지 분석
        - 코드 도달 가능성 분석 결과를 사용하여 메모리 접근 유형에 따라 프로세스의 도달 가능한 함수가 액세스 하는 전역 데이터만 읽기 / 쓰기 가능 등으로 표시
    2. Stack 또는 Heap의 경우
        - 분석을 통해 각 데이터 세그먼트의 위치 및 크기를 식별
3. 장치 접근성 분석
    - MMIO를 통해 주변 장치에 대한 메모리 접근을 식별
    - backward slicing 사용
    - 주변 장치 매핑 메모리 영역의 주소가 펌웨어 코드에 상수로 내장되어있으므로 액세스 정적 식별 가능

## B. Memory View Tailoring

- 메모리 뷰를 액세스 제어 규칙 목록으로 정의하며, 이 규칙은 액세스 가능한 메모리 영역의 위치와 권한 유형을 포함함
- MINION은 펌웨어 분석 결과를 기반으로 메모리 뷰를 구성함
- **Memory View Creation**
    - 메모리 뷰 : {주소 : 액세스 권한} 으로 되어 있는 배열(테이블)
    - 각 주소에 대해 모든 펌웨어 분석 결과 (Cx, Gx, Sx, Hx, Dx)를 열거하여 주소별 권한 합집합을 통해 VX가 만들어짐
- **Memory View Clustering**
    - 메모리 뷰를 강화하기 위해, MPU를 사용함
    - 그러나 MPU는 고정된 소수의 메모리 영역만을 지원함
        - 이를 해결하기 위해 유사한 개체를 그룹화하여 사용함
        - 접근 제어 규칙의 수를 최소화하는 알고리즘을 사용
    
    ![5](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/240275e1-3bc8-4132-a34f-5fa2a7a554cb)
    

## C. Memory View Enforcement

![6](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/089b0473-48ab-431e-b6d9-5323955bb4d7)

- 주어진 MPU access control rule을 기반으로, 펌웨어에 MINION system call을 삽입하여 다음과 같이 프로세스를 제어
    - *init*
        - RTOS가 장치 초기화를 마친 후 첫 번째 프로세스가 생성되기 직전 호출되는 시스템 콜
        - 호출 시, view switcher 영역이 비특권 모드에서 접근될 수 없도록 MPU 설정
    - *switch_view*
        - RTOS에서 Context switching 시 실시간 스케줄러에 의해 호출되는 시스템 콜
        - 새로운 프로세스가 자신의 view 내의 메모리 영역에만 접근할 수 있도록 MPU를 재구성
    - *read_scb & write_scb*
        - 비특권 모드에서 실행되는 소프트웨어 모듈이 System Control Block(SCB)에 안전하게 접근할 수 있도록 하는 시스템 콜
        - SCB의 경우 MPU에서 접근 권한을 허용하더라도 반드시 특권 모드에서만 접근 가능하기 때문에, 해당 시스템 콜을 통해  SCB 데이터를 주고 받음(read & write)

## D. Stack and Heap Allocation

- RTOS의 메모리 할당자는 수정되어 memory view를 준수하는 격리된 스택 및 힙 메모리를 가짐
- 새로운 프로세스가 생성되면 메모리 할당자는 shared memory 공간에 메모리 풀을 예약하여 동적 메모리 할당

# 6. Evaluation

![7](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/d5b5eec1-11ce-4acd-ac83-7807bcf74e8d)

- Nuttx OS를 사용하는 Drone인 PX4를 대상으로 성능 테스트 수행
- 모든 프로세스가 MINION을 사용하여 memory protection(노란색 그래프)을 켰을 때도, Deadline을 넘기지 않았음
- memory protection을 수행하지 않을 경우와 오버헤드가 크게 차이 나지 않음
    - **평균적으로 2% 정도의 오버헤드만을 가짐**

![8](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/e69c7eed-9ce9-448c-8afe-22ded1527187)

- 프로세스의 공격 표면 및 메모리 공간을 크게 줄였음
- MPU에서 제공하는 reigion(access rule)의 개수가 증가할 수록 메모리 공간은 더욱 감소함

![9](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/d480478a-f3f0-4a2d-a3ac-e9cc433d9b7d)

- 8개의 attack case를 모두 감지

---

# 개인적인 생각

첫 번째, 아키텍처가 static analysis의 결과에 크게 의존하기 때문에, 이에 따라 프로세스 메모리 공간 및 attack surface가 완벽히(x86 시스템의 MMU를 사용한 protection을 기준으로 생각했을 때) 커버되지 않는다는 문제점이 있을 것 같다. 이에 따른 문제가 분명히 있을텐데, evaluation에서 다루지 않은 점이 좀 아쉬웠다(본인들은 물론 안 밝히는 게 훨씬 이득이었겠지만).

두 번째, MPU access rule을 설정하기 위해 프로세스의 오브젝트들을 address와 permission이 유사한 것끼리 clustering을 수행하는데, 이 기준이 조금 애매한 것 같다. 프로세스의 주소가 산발적으로 배치 되어 있거나 중간지점에 배치되어 있는 경우, 어떻게 clustering 했느냐에 따라 protection 성능이 달라질텐데.. 궁금하다.