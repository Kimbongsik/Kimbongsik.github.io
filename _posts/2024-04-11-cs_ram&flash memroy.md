---
title: "[임베디드/컴퓨터구조] 메모리 별 특징(SRAM, DRAM, NAND, NOR)"
last_modified_at: 2024-04-11T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - cs
  - embedded

tags:
  - cs
  - embedded
---
컴퓨터에서 사용하는 메모리의 종류는 다양하다.

가장 단순한 구조는,

- CPU ← 램 ← 플래시 메모리

이렇지만, 속도와 성능 향상을 위해 컴퓨터에서는 메모리를 hierarchy 구조로 저장한다.

![1](https://github.com/Kimbongsik/Data-Structure/assets/63995044/03764a51-356d-4581-9a54-49078ec9ade5)

다음과 같이 계층 구조로 메모리를 구성하면,  속도 측면에서 성능이 크게 좋아진다.

왜냐하면 CPU보다 램의 속도가 더 느리고, 램보다 하드 디스크의 속도가 더 느리기 때문에 CPU에 가까운 쪽에 속도가 빠른 메모리를 배치하면 CPU가 데이터에 접근하는 속도를 높일 수 있다.

즉, 하드 디스크에 있는 데이터를 CPU로 전달할 때 속도가 느린 메모리부터 빠른 메모리 순으로 배치해두면 속도와 성능이 매우 좋아진다. 

위 그림과 같이 메모리를 배치 위치(내부/외부)에 따른 구분으로 나눌 수도 있지만,
휘발성 여부로 나눌 수 있다는 것도 우리는 안다.

- 휘발성 메모리: register, cache, main memory
- 비휘발성 메모리: hard disk

CD, HDD, SSD와 같이 컴퓨터가 꺼졌다 켜져도 데이터가 남아 있어야 하는 메모리는 비휘발성, RAM과 같이 연산을 위해 데이터가 올려졌다가 컴퓨터가 꺼지면 데이터가 사라지는 메모리는 휘발성 메모리다.

현존하는 메모리 반도체는 SRAM, DRAM 등 여러가지 종류가 있는데, 다음을 보고 어떤 반도체가 어떤 용도로 쓰일지 추측해보자.

|  | SRAM | DRAM | NAND | NOR |
| --- | --- | --- | --- | --- |
| 휘발성 여부 | 휘발성 | 휘발성 | 비휘발성 | 비휘발성 |
| 속도 | 매우 빠름 | 빠름 | 느림 | 느림 |
| 데이터 저장 방법 | Flip Flop(Latch) | Capacitor | Floating gate | Floating gate |
| 용량 | 작음 | 중간 | 큼 | 큼 |
| 집적도 | 낮음 | 높음 | 높음 | 높음 |

## SRAM vs DRAM(with DDR)

SRAM은 Static RAM, DRAM은 Dynamic RAM으로,

SRAM은 flip-flop으로 데이터를 저장하기 때문에 전류 신호가 오기 전에는 상태가 변하지 않는다. 그래서 Static. 

→ 메모리만 기억하면 되므로 속도가 빠르다

→ 하지만 복잡해서.. 집적도가 낮다.

→ 그래서 용량도 낮다.

DRAM은 커패시터로 저장하기 때문에 일정 간격으로 충전을 시켜줘야 한다(커패시터 특성 상 시간이 지나면 방전됨). 그래서 Refresh 회로가 추가적으로 달려 있다. 그래서 Dynamic.

→ 이를 계속 신경 써줘야 하기 때문에 속도가 느리다

→ 하지만 단순해서.. 집적도가 높다

→ 대신 용량이 크다 

DRAM에는 또 두 가지 종류가 있는데,

- SDRAM: Synchronous DRAM으로, system bus와 동기를 맞추어 동작하므로 효율이 높은 메모리
- DDR SDRAM: Synchronous Double Data Rate으로, clock의 falling edge와 rising edge 모두에서 동작을 하므로 데이터가 두 배로 빨리 전송될 수 있음
    - DDR2 SDARAM : 외부 데이터 보스를 DDR SDRAM의 두 배 빠른 속도로 작동. 내부 클럭 속도는 DDR과 동일하나, 향상된 I/O 버스 신호로 인해 전송 속도가 2배. (전압 1.8V, 전송 속도 533~800 MT/s)
    - DDR3 SDRAM : DDR2 모듈에 비해 40%의 전력소비를 낮춰 낮은 전류 및 전압을 제공하나 전송 속도는 더 빠름 (동일 전압, 전송 속도 800~1600 MT/s)
    - DDR4 SDRAM : 더 낮은 전압, 더 높은 속도 (전압 1.2V, 전송 속도 2133~3200 MT/s) + DBU(Data Bus Inversion), CRC (Cyclic Redundancy Check) 등의 기능 탑재
    

## NAND Flash & NOR Flash

NAND Flash와 NOR Flash를 가르는 차이는 ‘반도체 셀의 연결 방법’이다. 

**NAND Flash는 셀이 직렬로 연결**되어 있으며, **NOR Flash는 병렬로 연결**되어 있다. 이 차이로 인해 두 플래시의 용도 및 특징이 달라지게 된다. 

NAND Flash는 셀이 직렬 형태이기 때문에,

1. Random Access가 불가능하며,
2. 따라서 각 셀에서 순차적으로 데이터를 읽어내야 하기 때문에 NOR Flash에 비해 Read 속도가 느리다.
3. 그러나 메모리 블록이 나누어져 있어 Write / Erase 속도는 더 빠르다.

NOR Flash는 셀이 병렬 형태이기 때문에,

1. **Random Access가 가능하며,**
2. 때문에 Read 속도가 NAND Flash에 비해 느리다.
3. 셀이 병렬 형태로 이어져 있으므로, 각 셀을 개별적으로 접근하기 위한 전극이 별도로 필요하게 되어 NAND Flash에 비해 집적도는 낮다. 즉, Cell size가 크다.

추가적으로 말하자면, NOR Flash는 Random Access가 가능하다는 특성 때문에 임베디드 시스템에서 한때 자주 사용되었다.

어떻게 사용되었느냐 하면 NOR Flash에 Code를 넣고, 램이 아닌 Flash에서 직접 코드를 실행시키는 방식으로 시스템을 설계한 것인데, 이를 **XIP(eXecution In Place)**라고 한다. 코드와 데이터의 크기가 작은 시스템의 경우 사용하기 용이하며, RAM에 데이터를 올리지 않더라도 Flash에서 코드 실행이 가능했기 때문에 개발 편의성이 좋다. 

물론 요즘은 펌웨어의 크기가 계속 커지다 보니, NAND Flash에 코드를 저장하고, 해당 코드를 RAM(SDRAM)에 복사하여 RAM에서 데이터를 실행하는 형식도 많다.

---

참고 자료 :

https://m.blog.naver.com/ki630808/221621584087

https://yeji1214.tistory.com/20

https://amanan1004.tistory.com/27