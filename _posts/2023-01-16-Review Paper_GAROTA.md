---
title: "GAROTA: Generalized Active Root-Of-Trust Architecture (for Tiny Embedded Devices)"
last_modified_at: 2023-01-16T16:20:02-05:00
categories:
  - papers

tags:
  - Usenix
  - embedded
  - 임베디드
  - ROT
  - security
  - 보안 
---


*용어

- RoT(Root of Trust): OS에서 항상 신뢰하는 컴퓨팅 모듈의 기능 집합으로, 변경이 불가능하고 컴퓨팅 시스템의 모든 보안 작업이 의존하는 신뢰할만한 장치
- TCB(Trusted Computing Base): 시스템에 안전한 환경을 제공하기 위해 결합된 모든 컴퓨터 시스템의 하드웨어, 펌웨어 및 소프트웨어 구성 요소 (ex: ARM의 TrustZone)
- DMEM: Data memory 휘발성(
- PMEM: Program memory 비휘발성(스택, 힙 저장)

*읽기 전 고려사항
- 어떻게 TEE를 안 쓰는가?
- 왜 RoT라고 하는가? 이 안전한 영역이 어떤 방식으로 보장되는가?

# Abstract

**>연구 배경**

- Low-end Embedded device는 정교한 보안 서비스를 구현하기 위한 리소스가 부족한 환경으로 인해 보안 유지가 어려움
- 소프트웨어 상태 및 런타임 무결성에 대한 원격 확인과 같은 서비스 활성화를 위한 RoT(Roots-of-Trust)가 제안됨
- 해당 RoT는 소프트웨어 업데이트 등 특정 작업이 특정 장치에서 수행되었는지의 여부를 증명
- 그러나 원하는 행동이 수행될 것이라고 보장 불가 → 장치 제어 malware는 수신된 명령 및 트리거 이벤트를 무시/폐기하여 RoT에 대한 access를 쉽게 차단 가능하기 때문
- RoT의 대부분은 TEE 형태의 광범위한 하드웨어 지원에 의존하며 이는 일반적으로 저가형 장치에는 너무 많은 비용이 듦

**>제시**

- Low-end MCU를 위한 minimal active RoT 설계
- malware가 있는 경우 조치를 보장하는 기능
- GAROTA 구현
- GAROTA 보안 이점
- sensing hardware, network events and timers

# Introduction

- 기존 secure한 low-end MCU(micro controller unit)를 위한 아키텍쳐는 원격 증명, 원격 소프트웨어, 제어 흐름 및 데이터 흐름 증명, 원격 소프트웨어 업데이터, 메모리 삭제 등등 수동적으로 작동한다. → 잠재적인 MCU의 손상이 있음에도 불구하고 위조 불가능하다는 증명을 제공하고 있음(?)
- 왜 수동적이냐?
    - 원격 디바이스에 대한 무결성 위반은 감지 가능하지만, 보안/안전을 위해 수행되는 task가 바르게 수행되었는지에 대한 것은 보장하지 않기 때문
    - 맬웨어가 RoT에 대한 모든 요청을 가로채거나 무시/폐기할 수 있으므로, 보안 기능이 트리거 되는 것을 막을 수 있음.
- 손상된 장치를 복구하는 방법
    - 각 장치에 대한 물리적이고 직접적인 접근(느리고 파괴적)을 수정
- GAROTA
    - **런타임에서의 트리거 구성(인터럽트 테이블, 인터럽트 설정 레지스터 등)을 보호하기 위해 신뢰할 수 있는 하드웨어를 추가한다. 이는 소프트웨어에서 수정하거나 비활성화가 불가능하다.**
    - 손상될 수 있는 상태인 MCU의 하드웨어 주변장치에서 캡쳐한 임의의 이벤트를 기반으로 신뢰할 수 있고 안전에 중요한 작업의 보장된 실행을 trigger 할 수 있는 아키텍쳐
    - 수정되지 않은 MCU에서 인터럽트를 유발하는 모든 하드웨어 이벤트는 GAROTA에서 신뢰할 수 있는 소프트웨어의 보장된 실행을 트리거할 수 있음
    - 이전 연구는 기존에 존재하는 하드웨어에 의존적이기 때문에 하드웨어 장치를 바꾸지 않아도 된다는 장점이 있음
    - 반면, GAROTA는 소형 하드웨어 디자인 기반이며, 새로운 application이 가능하고 low-end 리소스 제약을 가지고 있는 MCU에 적용 가능
    - **Trigger: GAROTA RoT가 MCU에서 실행을 인계받는 이벤트. TCB가 실행되도록 구성할 수 있는 특정 이벤트.**
    - Guaranteed Triggering
        
        특정 관심 이벤트가 항상 GAROTA RoT 실행을 트리거 하도록 함
        
    - Re-Triggering on Failure
        
        RoT 실행이 불법적으로 중단되는 경우(실행 무결성, 전원 오류, 재설정 시도 등) MCU가 재설정되고 RoT가 첫번째가 되도록 보장
        
    - 제안
        - GPIO-TCB: safety-critical sensor/actuator hybrid 등, 센싱되는 데이터 양이 임계값(threshold)을 초과하는 경우 알람을 울리도록 보장
        - Timer-TCB: safety-critical 작업이 사전 정의된 real time system이 주기적으로 실행 되도록 보장
        - NetTCB: 네트워크를 통해 수신된 명령을 실행하도록 보장하는 trusted component. MCU의 멀웨어가 RoT로 향하는 명령을 가로채거나 폐기하는 것을 방지
        
        → 모두 RoT 작업 자체를 신뢰하는 한 MCU 소프트웨어 상태가 완전히 손상된 경우에도 보장이 유지됨.
        

# Scope

- 프로그램 및 데이터 메모리가 몇 KBytes 정도 되는 low power single core platform 대상(Atmel AVR ATmega, TI MSP430)
    - 8, 16-bit CPU
    - 클럭 주파수 16MHz
    - SRAM: 4~16KB 크기의 데이터 메모리로 사용, 나머지 주소 공간은 프로그램 메모리로 사용
    - bare metal 위에서 소프트웨어를 구동
    - 가상 메모리를 지원하는 MMU 없음
    

# GAROTA Overview

![1](https://user-images.githubusercontent.com/63995044/214065706-3ce2c5af-38be-4dae-88b3-28062ef1a438.png)<img src='https://user-images.githubusercontent.com/63995044/214065706-3ce2c5af-38be-4dae-88b3-28062ef1a438.png'>

- 목표
    - 신뢰할 수 있는 소프트웨어 실행 파일로 구현된 미리 정의된 기능(F)의 이벤트적 실행을 보장함
    - 이 실행 파일을 GAROTA TCB(Trusted Computing Base): GAROTA 신뢰 컴퓨팅 기반 이라고 함
- 동작 방식
    - 외부 입력(센서 입력, 온도 감지 등), 타이머, 네트워크 패킷 도착 등의 하드웨어 이벤트가 발생하면 F의 실행을 보장하는 GAROTA에서 사용하는 인터럽트를 발생시킴
    - 트리거 및 TCB 구현은 구성 가능하고, 이러한 초기 설정이 초기 배포 전에 안전하게 수행된다고 가정
    - 런타임 시 GAROTA는 불법적인 수정으로부터 초기 구성을 보호 → 올바른 트리거 동작을 보장
    - 인터럽트 구성 레지스터, 인터럽트 핸들러 및 인터럽트 벡터 보존 포함
    
    **→ 트리거가 항상 TCB를 호출하도록 보장**
    
    - TCB 코드 자체가 변조될 수 있으므로, 권한이 없거나 신뢰할 수 없는 프로그램이 tcb 코드, 즉 해당 코드를 저장하기 위해 예약된 프로그램 메모리 영역을 수정하지 못하도록 하는 런타임 보호 기능을 제공
    - TCB 모니터링
        - Atomicity(원자성): TCB의 첫 번째 명령어부터 마지막 명령어까지 atomic하게 실행된다. 즉, 중간에 종료되지 않는다.
        - Non-malleability(가단성: 금속을 두드려 늘일 수 있는 성질): 실행 도중 DMEM은 수정될 수 없다. TCB 코드 외에는. (ex: 다른 소프트웨어 또는 DMA 컨트롤러에 의한 수정 불가)
    
    → 해당 두 가지 특성은 MCU 내부의 잠재적인 맬웨어가 TCB 실행을 방해(tamper)하지 않도록 보장
    
    - GAROTA는 실행을 모니터링 하고, 속성(원자성, 비가단성, TCB 코드 변조 및 잘못된 트리거 포함)의 위반을 감지할 때마다 즉시 MCU를 신뢰할 수 있는 기본 상태로 reset. TCB 코드는 실행의 첫 번째가 됨.
    - TCB 기능 및 실행에 대한 간섭 시도 시, TCB 복구 및 재실행을 하여 허가되지 않는 앱들은 간섭을  할 수 없도록 보장
    - Trigger 구성과 TCB 구현은 모두 TCB 자체 내에서 업데이트가 수행되는 한 런타임에 업데이트 가능
    - GAROTA의 하위 속성은 별도의 하위 모듈로 구현되고 개별적으로 최적화 됨
    - 해당 하위 모듈은 모델 확인 기반 형식 검증과 LTL 컴퓨터 확인 증명의 조합을 사용해 안전하게 구성되고 표시됨
    - GAROTA 모듈식 디자인은 검증 가능성과 최소화를 가능하게 해 하드웨어 오버가 적음
        
    ![2](https://user-images.githubusercontent.com/63995044/214065716-f1f38508-4d02-47ec-984d-954586179221.png)<p alighn="center"><img src='https://user-images.githubusercontent.com/63995044/214065716-f1f38508-4d02-47ec-984d-954586179221.png'></p>
        
    - 그림2: GAROTA는 필수 보안 속성에 대한 위반을 감지하기 위해 일련의 CPU신호를 모니터링하는 하드웨어 구성 요소로 구현됨
    - 동작이나 명령어 세트를 수정하는 등 CPU 코어 구현을 방해하지 않음
    - 범용 FPGA를 사용해 low-end MCU MSP430에서 GAROTA를 합성하고 오버헤드를 보고

# 4. Detail

## 4.1 LTL(Linear Temporal Logic) & Verification Approach

- LTL: 선형 시제 논리 (논리 연산~)

![3](https://user-images.githubusercontent.com/63995044/214065723-62f30a63-419f-47b5-b041-aa4464f0f3a8.png)<p alighn="center"><img src='https://user-images.githubusercontent.com/63995044/214065723-62f30a63-419f-47b5-b041-aa4464f0f3a8.png'></p>

## 4.2 Notation, Machine Model, & Assumptions

### 4.2.1 CPU Hardware signals

- GAROTA는 CPU의 명령어 set을 수정하거나 확인하지 않음. CPU와 병렬로 처리되는(run) 독립적인 하드웨어 모듈로 구현되고 하드웨어에서 필요한 보장을 시행한다고 가정

- H1 - PC: CPU가 실행 중인 명령어의 주소를 저장
- H2 - Memory Address: Daddr(data address signal)은 CPU가 메모리를 읽고 쓸 대 해당 메모리 주소를 저장. 읽기 접근 시 Ren(Read enable bit)는 세팅 되어야 하며 write access 시엔 Wen 비트가 세팅되어야 함
- H3 - DMA: DMA 컨트롤러가 주 시스템 메모리에 액세스 하려고 시도할 때마다 DMA 주소 신호(DMAaddr)는 액세스 중인 메모리 위치의 주소를 반영하고 DMA 활성 빝트(DMAen)를 설정해야 함. DMA는 DMAen이 꺼져 있을 때(0) 메모리에 액세스할 수 없음
- H4 - MCU Reset: 정상적인 소프트웨어를 다시 시작하기 전에 모든 레지스터가 0으로 설정됨. (하드웨어에서 MCU에 의해). 실행이 다시 시작되면 PC는 INIT이라고 하는 프로그램 메모리의 부팅 섹션에 있는 첫번째 명령어를 가리키도록 설정됨. DMA 컨트롤러 및 이전 구성을 재설정함. (DMA 동작은 런타임 시 소프트웨어에 의해 구성되며, 부팅 및 reset 후 기본적으로 비활성화 되어 있음)
- H5 - Interrupt: 인터럽트가 발생할 때마다 irq 신호가 설정됨. gie라는 1비트 시그널은 현재 활성화 되어 있는지 여부를 반영함. 부팅 및 reset 후 기본적으로 비활성화 되어 있음

### 4.2.2 Memory: Layout & Initial Configuration

- M1 - PMEM: 전체 PMEM 주소 공간에 해당하며 런타임 시 PC는 실행 중인 명령을 저장하는 PMEM 주소를 가리킴
- M2 - INIT: MCU 부팅 세그먼트, 즉 MCU가 부팅될 때마다 또는 재설정 후 실행되는 첫 번째 소프트웨어를 포함하는 PMEM의 섹션
- M3 - TCB: GAROTA 신뢰 코드ㅡ 용으로 예약된 PMEM 섹션, 즉 F. TCB는 INIT 바로 뒤에 위치하며 INIT이 성공적으로 완료된 후 실행되는 첫 번째 소프트웨어
- M4 - IRQ 테이블 및 핸들러: IRQ 테이블은 PMEM에 위치하며 인터럽트 핸들러의 주소에 대한 포인터를 저장. 인터럽트가 발생하면 MCU 하드웨어가 해당 핸들러 루틴으로 점프함. 해당 루틴의 주소는 특정 이넡럽트에 해당하는 IRQ 테이블 고정 인텍스에 의해 지정됨. 핸들러 루틴은 PMEM에도 저장되는 코드 세그먼트임
- M5 - IRQ(cfg): 런타임 시 개별 인터럽트의 특정 동작을 구성하는 데 사용되는 DMEM의 레지스터 집합

→ 해당 메모리 구성은 GAROTA 검증 하드웨어 모듈에 의해 명시적으로 보호되지 않는 한 런타임에 변경될 수 있음

### 4.2.3 Initial Trigger Configuration

- T1 - trigger: GAROTA trigger는 MCU predeployment-time에 IRQ 테이블의 항목과 TCB의 첫 번째 명령으로 점프하도록 각 핸들러를 설정하고 원하는 인터럽트 매개변수로 IRQcfg의 레지스터를 구성하여 구성된다. (원하는 트리거 동작 예제: 섹션 5) 따라서 초기 구성이 유지 되는 한, 트리거 이벤트로 인해 TCB 코드가 실행됨

→ 초기 트리거 구성은 일반적인 임베디드 프로그램의 일반 인터럽트 구성과 크게 다르지 않음. 인터럽트가 해당 핸들러 진입점을 올바르게 가리켜야 하는 것처럼, IRQ-Table의 트리거 인덱스는 GAROTA TCB 법적 진입점을 올바르게 가리켜야 함.

- 예를 들어 타이머 기반 트리거를 초기에 구성하기 위해 각 하드웨어 타이머에 해당하는 IRQ-Table의 주소는 TCBmin을 가리키도록 설정되고 IRQcfg 레지스터는 타이머 데드라인과 원하는 인터럽트 기간을 정의하도록 설정됨

### 4.2.4 Adversarial Model

- 적(Adv): 하드웨어 적용 액세스 제어 규칙에 의해 명시적으로 보호되지 않는 모든 메모리에서 읽고 쓸 수 있음. MCU의 DMA 컨트롤러를 완전히 제어(CPU를 거치지 않고 메모리에 access 할 수 있음).
- Pysical Attacks: GAROTA 지원 X
- Network Dos Attacks: Adv가 MCU에서, 혹은 -로부터 트래픽 및 통신을 방해한다.

→ 네트워크 트래픽 모니터링과 같은 직교 기술로 네트워크 Dos 공격을 탐지할 수 있음. 탐지되면 이러한 공격을 완화하기 위해 대역 외 조치를 취할 수 있음. GAROTA는 명령이 도착하고 트리거가 발생하더라도 MCU를 감염시키고 명령/트리거 이벤트를 무시하는 맬웨어에 관심이 있는데, *GAROTA NetTCB 사용 사례에서 네트워크 Dos 모니터링은 패킷을 MCU로 전달하기 위한 보완 조치이며, GAROTA는 수신된 명령이 실제로 처리되었는지 보장할 수 있음.

- Correctness of TCB’s Executable: 많은 응용 프로그램에서 F 코드는 최소이다(섹션 5의 예시). F의 정확성은 locally 하게 보장될 필요도 없다. 임베디드 어플리케이션은 PC와 같은 더 상위 디바이스에서 개발되기 때문에 구현 버그 탐지 등을 위해 퍼징,, 등등을 할 수 있다~ 그래서 GAROTA TCB에 로드하기 전에 오프라인에서 수행할 수 있는 모둔 문제를 다 테스트 해보면 된다~

### 4.2.5 Machine Model(Formally)

- LTL을 사용해 MCU 머신 모델의 subset을 구성

*삼지창: a (3C) b : a는 b의 원소

*이지창: 부분집합

![4](https://user-images.githubusercontent.com/63995044/214065731-b3671b7a-d2c2-4ee6-a5e6-c7cc8557ff01.png)<p alighn="center"><img src='https://user-images.githubusercontent.com/63995044/214065731-b3671b7a-d2c2-4ee6-a5e6-c7cc8557ff01.png'></p>

(1): 메모리 수정(주소: X)에 대한 수정은 CPU 또는 DMA를 통해 이루어짐

-CPU에 의한 수정: (Wen=1 ^ Daddr=X)  Wen: Write enable bit

-DMA에 의한 수정: (DMAen = 1 ^ DMAaddr )

(2): 

### 4.3 GAROTA End-To-End Goals Formally

### 4.4 GAROTA Sub-Properties

LTL 8은 신뢰할 수 없는 애플리케이션에 의해 인터럽트가 전역적으로 비활성화될 수 없도록 강제합니다. 각 트리거는 인터럽트를 기반으로 하므로 모든 인터럽트를 비활성화하면 신뢰할 수 없는 소프트웨어가 트리거 자체를 비활성화하여 GAROTA의 활성 동작을 비활성화할 수 있습니다.

# 5. Sample Applications

GAROTA의 일반성을 입증하기 위해 세 가지 구체적인 예의 프로토타입을 제작함.