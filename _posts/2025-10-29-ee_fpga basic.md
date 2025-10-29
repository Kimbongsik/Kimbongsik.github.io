---
title: "[전기/전자] FPGA(Field Programmable Gate Array)"
last_modified_at: 2025-10-29T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - ee

tags:
  - ee
---
## 1. FPGA란?

- ASIC(Application Specific Integrated Circuits)과 달리, 수정 가능한 반도체임
    - ASIC은 회로 설계 > 반도체 공정(파운드리) 까지의 일련의 과정을 거쳐서 개발되는 회로임
- FPGA는 프로그래밍이 가능하다는 특성으로 인해 반도체 생산 공정을 생략할 수 있음
    - 생산 볼륨이 작거나 대규모 생산이 비효율적인 경우 FPGA를 선택하는 전략을 선택할 수 있음
    - 설계한 하드웨어에 버그가 있거나 기능 추가를 원하는 경우 언제든 새로운 로직을 로드할 수 있음
    - 로직 프로토타입(prototyping)용으로도 사용됨

## 2. 구성

### **Configurable Logic Block(CLB)**

- 디지털 회로를 원하는 대로 구현할 수 있도록 하는 블록
- 내부적으로 슬라이스(slice)라는 단위로 구성되어 있어 높은 유연성을 보장함
    - 논리 연산과 데이터 처리를 동시에 수행할 수 있는 병렬 구조 제공
    - 한 개의 연산이 전체 블록을 점유하지 않으므로, 동시에 여러 연산을 수행할 수 있음
- 슬라이스는 아래와 같은 요소로 구성되어 있음
    - Flip-Flop
    - LUT
    - Multiplexer
    - Carry chain block

> **Flip-Flop**
> 
- 데이터 저장 및 회로 간 데이터 동기화
- 클럭 신호에 맞춰 입력 데이터를 저장한 뒤 이를 후속 연산에 사용하거나 다음 단계의 논리 회로로 전달
- 파이프라인 구조 설계에서 데이터 일관성 + 처리 속도 유지에 사용

> **LUT(Look Up Table)**
> 
- 입력 신호의 조합에 따라 미리 정해진 결과를 출력
- 기본 연산부터 계산까지 LUT에 저장된 값으로 처리하여, 작은 메모리처럼 설계자가 원하는 논리 동작을 빠르고 효율적으로 구현할 수 있음
- SRAM 셀과 Mux를 통해, 여러 개의 입력을 받아 한 비트의 출력을 만들어 줌

<img width="369" height="468" alt="Image" src="https://github.com/user-attachments/assets/7bb80597-196c-4644-b078-c70298367125" />

> **Multiplexer**
> 
- 여러 입력 신호 중 하나를 선택해 출력하는 장치
- 이를 사용해 논리 연산 결과를 선택하거나 특정 데이터 경로를 활성화할 수 있음
- 두 개의 데이터 경로 중 하나를 선택해 출력하거나, 조건에 따라 신호를 전환하는 데 활용

> **Carry Chain Block**
> 
- 비트 연산 시 자리 올림(carry)을 빠르게 처리 및 계산하기 위해 설계된 특수 하드웨어
- 큰 숫자의 덧셈 연산에서 carry가 다른 경로를 거치지 않고 바로 다음 단계로 전달되도록 설계

## DSP(Digital Signal Processing)

- FPGA 내부에서 연산 속도를 극대화 하기 위한 블록
- ALU(Arthmetic Logic Unit)과 유사한 역할을 수행하며, 산술 연산을 하드웨어적으로 처리하여 연산 속도를 매우 향상 시킴
- 덧셈기, 뺄셈기, 곱셈기, 누산기

## 3. 메모리

> 개념
> 
- 6-input LUT를 사용한다고 했을 때, 이는 64비트를 저장할 수 있음
- 한 개의 슬라이스에는 8개의 6-input LUT가 있으므로 8 * 64 == 512bit를 저장할 수 있음

> BRAM(Block RAM)
> 
- On-Chip 메모리로, FPGA 패브릭에 있는 듀얼 포트 램임
    - 즉, 1 cycle에 2개의 주소로부터 값을 읽거나 쓸 수 있음
- 보통 18K bit , 36K 비트를 저장할 수 있음
    - 36000 bit == 4500 pixel == 60 * 60 이미지

> URAM(Ultra RAM)
> 
- Xilinx FPGA의 UltraScale+ 이상의 제품군에 있는 듀얼 포트 온칩 메모리
- BRAM보다 8배 더 저장할 수 있음

> Shift Register
> 
- 레지스터를 연결해둔 것으로, 이전 입력 값을 저장하고 재사용 하는 데 사용됨



-참고자료: https://wikidocs.net/89806

