---
title: "[임베디드] ARM 프로세서 Modes & Vector Table"
last_modified_at: 2023-11-21T16:20:02-05:00
categories:
  - embedded

tags:
  - embedded
  - ARM
---

<aside>
💡 본 글은 책 "임베디드 레시피" by 히언 을 참고하여 작성된 글입니다.

</aside>

<br>

# ARM Exceptions and Modes

- ARM은 시스템 접근 용도에 따른 7개의 모드를 제공
    - **USR: Normal Program execution mode**
    - **SYS: Run privileged os tasks**
    - **FIQ: When a high priority(fast) interrupt is raised**
        - 인터럽트 중 Fast Interrupt 발생 시 진입
    - **IRQ: When a low priority(normal) interrupt is raised**
        - 하드웨어 인터럽트가 발생하여 ARM Core에 알려주면 진입
    - **SVC: A protected mode for the os, entered when a SWI instruction is executed**
        - 부팅 / 재부팅이 일어난 경우 진입
        - SWI: software interrupt로, 다른 예외처리들은 하드웨어적으로 발생하여 exception 발생 시 벡터 테이블로 이동하지만, SWI는 프로그래머가 코드 상에서 발생시키고 싶은 위치에 발생시켜 벡터 테이블로 jump 함
    - **ABT: Used to handle memory access viloations**
        - 접근하려는 주소가 접근 불가능한 주소이거나, 명령어 Fetch에 실패한 경우 진입. MMU 및 MPU를 사용할 경우 Access Protection이 걸려 있는 주소를 함부로 접근하려고 했을 때 ABT exception 발생
    - **UND: Used to handle undefined instructions**
        - 명령어 Decode 시, ARM이 모르는 것일 경우 진입. Memory Corruption 발생 시 진입.
            - 이를 활용하여 ARM이 사용하지 않는 코드를 삽입하고 UND Vector 주소로 jump 하게 하여 디버깅 Code 등의 의도된 일을 실행하게 할 수 있음

![_1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/ca8bfc6b-9078-4117-a473-e0f1d96c6bdc)

(이미지 출처: 임베디드 레시피, 히언)


<br>

# Exception Vector Table

- Mode 진입 방법에는 두 가지 방법이 있음
    - Priviliged Mode(USER Mode를 제외한 모든 모드)는 자기 마음대로 mode 변환이 가능( Priviliged Mode → User Mode (O), User mode → Priviliged Mode (X), Priviliged Mode ↔ Priviliged Mode (O))
    - 그 외에 하드웨어적으로 특정 Mode에 진입할 수 있음
        
        → 이 하드웨어적 Mode 변경을 위한 사건이 “**Exception**”.
        

- Exception 동작 개요
    - Exception 종류에 해당하는 Mode에 진입
    - 해당 상황을 처리하는 프로그램이 저장된 주소로 PC register jump
    - Exception에 대한 동작을 하드웨어적으로 처리

위와 같이 동작하기 위해서는 Exception이 발생했을 때 Jump 해야하는 주소들이 어딘가에 매핑되어 있어야 한다.

ARM사에서는 이러한 주소들을 Exception Table이라고 하여 정의해놓고, 0x0번지부터 순서대로 저장한다. 

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/3659a25b-a4e5-4f83-af83-7ec1b73d8b2e)

(이미지 출처: 임베디드 레시피, 히언)

예를 들어, SVC mode는 ARM 프로세서에 전원을 인가하거나 reset을 시키면 SVC 모드로 진입하면서 PC를 0x0번지로 jump 시킨다. 마찬가지로 IRQ와 FIQ 모드는 인터럽트가 발생하면 각 해당 모드로 진입하면서 PC를 0x18 또는 0x1C로 점프시킨다. 다른 모드들도 마찬가지이다.

Vector Table의 각 주소에는 Reset 동작을 처리하는 프로그램의 주소값이 저장되어 있다. PC를 Vector Table의 각 주소로 setting 하면, 해당 지점부터 소프트웨어가 실행되며, 정해진 주소의 프로그램을 하드웨어적으로 수행한다.

테이블을 자세히 보면, USER mode와 SYS mode로 진입하는 Exception이 없다는 걸 확인할 수 있는데, 이 말인 즉 User mode에서는 Exception 없이 Privilged Mode에 진입할 수 없다는 뜻이다. System mode에서는 User mode로 진입이 가능하다. 

<br>

# Exception 우선 순위

동시에 Exception이 발생하게 됐을 때, 시스템을 보호하기 위해 우선순위 순서대로 Exception을 처리한다. 보통 시스템에 직접적으로 악영향(?)을 주는 순서로 우선순위가 배치되어 있다. 우선순위는 아래와 같다.

1. Reset
    - Reset이 들어오면 당연히 컴퓨터가 꺼지므로..
2. Data ABT
    - 데이터를 읽어올 수 없으면 아무런 소용이 없으므로 우선 순위가 높다.
3. FIQ
4. IRQ
5. Prefetch ABT
6. UND
    - 다른 Coprocessor나 주변 기기를 직접 어셈블리어로 제어할 때 일부러 해당 exception을 발생시켜 핸들링하기도 하므로, 해당 모드의 우선 순위는 낮다.
    - Memory Corruption이 일어나서 발생하는 경우도 있다.
7. SWI
    - 프로그래머가 의도적으로 발생시키는 exception이므로 우선 순위가 제일 낮다

<br>

# Exception 처리 과정

Exception은 말 그대로 프로그램 실행 중 오류나 외부 요청 등의 예외가 들어와 처리하는 과정으로, 해당 Exception 동작을 수행한 후 원래 진행 중이었던 프로그램 동작으로 돌아와야 한다.

그러기 위해서는 벡터 테이블로 점프하기 전에 현재 실행 중이었던 Mode 정보와, Context, 그리고 수행 중이던 주소를 저장해야 한다.

Mode 정보는 CPSR(Current Program Status Register)이라는 레지스터에 저장되며, Context(Register 상태)는 Stack 메모리에, 돌아올 주소는 r14(LR)에 저장된다.

![3](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/e9cefe65-50fc-4697-b12a-f72cbf199260)

(이미지 출처: https://computersource.tistory.com/72)

따라서 A mode에서 B mode로 전환한 후 다시 A mode로 복귀하는 과정을 그려보면,

1. CPSR(현재 mode 정보를 저장)을 SPSR_B에 복사
2. CPSR의 mode 정보를 B mode로 변경, stack pointer(r13) 변경
3. IRQ disable, ARM mode로 변경
    - IRQ disable(인터럽트 불가): CPSR의 I bit를 1로 변경
    - ARM mode: CPSR의 T bit를 0으로 변경
        - 즉, Exception Vector는 무조건 32bit ARM mode로 프로그래밍 해야 한다.
4. r14_B(LR)에 현재 PC 레지스터 값 저장
5. PC에 B mode의 Exception Vector 주소 저장 ————— 여기까지는 하드웨어적 처리.
6. R0~R12를 R13_B(SP)이 가리키는 스택 주소에 저장 
7. 돌아갈 주소 보정 (sub lr, lr #4. 인터럽트가 걸린 순간 r14 := PC를 수행했으므로 pipe line에 의해 2개의 opcode가 이미 진행됨).
8. CPSR에 SPSR_B을 넣어 이전 mode인 A mode로 복원
9. Stack에 저장했던 레지스터값들 복원
10. PC에 r14_B 값을 넣어 A mode에서 처리하던 곳으로 복원

<br>

# Mode 사용 장점

각 모드마다 Register Set을 가지고 있어 mode 전환이 빠르다.

예를 들어 User mode에서 Interrupt가 발생하여 IRQ mode로 전환이 되면, User mode에서 쓰던 레지스터들을 모두 저장할 필요 없이 IRQ mode로 전환하면 된다.

모드마다 전체 레지스터를 독점적으로 쓰는 것은 아니고, R0-R7는 공통적으로 사용되며 아래 그림에서 삼각형 표시가 되어 있는 banked register 들은 각 mode 마다 따로 존재한다.

FIQ에 banked register가 많은 이유는, 빨라야 하기 때문이다… ㅎㅎ

다른 모드로 전환할 때 레지스터에 있는 데이터들을 stack에 저장해둬야 하는데, FIQ 모드로 전환될 경우 그럴 필요 없이 전용 banked register를 사용하면 되므로 모드 전환이 빠르게 이루어진다.

이렇게 해서 ARM에서 사용하는 총 레지스터는 37개이다.

![4](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/6cd2d7ce-1b73-4741-8e2b-18b4105688e3)

(이미지 출처: 임베디드 레시피, 히언)