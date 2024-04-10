---
title: "[임베디드] SPI 개념 및 Register"
last_modified_at: 2024-03-29T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
지난 번 UART 통신 포스팅에 이어, 이번에는 SPI 통신 방식에 대해 정리해본다.

왜냐하면 곧 써야 하거든.. 어디에 쓸 건지는 다음에 공개한다.

# SPI 통신이란

> **정의**

SPI(Serial Perihperal Interface) 통신은 시리얼 통신 인터페이스이며, MCU나 SD 카드 등의 소형 주변 장치 사이에 데이터를 전송하기 위해 사용된다. 즉, 통신 거리는 짧은 편이다.

즉, IC(칩)과 칩 사이 데이터를 주고받기 위한 통신 방법 중 하나로, **1 : N 통신**을 지원하며, **동기식 통신**이며, **Full-duplex 통신**이 가능하다. 1bit 씩 송신하고 동시에 1bit 씩 수신하며, 실질적으로 16 bit 순환 shift register 형태로 데이터를 전송한다.

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/5e7157df-e7ac-4c4b-bcdd-1eac70fb55ac)

> **Master & Slave 통신**

SPI는 1개의 마스터 기기와 N개의 슬레이브 기기를 연결하여 사용한다.

사용하는 신호는 아래와 같다.

- **SCLK(SCK)** : 클럭 전송을 위한 단자. Master → Slave로 클럭 전송
- **MOSI** : Master Ouput Slave Input. Master → Slave로 데이터를 전송할 때 쓰는 단자이다.
- **MISO** : Master Input Slave Output. Slave → Master로 데이터를 전송할 때 쓰는 단자이다. MOSI를 통해 Slave에 명령 데이터가 들어오면 MISO를 통해 응답 데이터를 송신한다.
- **CS(=SS)** : Chip Select 혹은, Slave Select. Master 디바이스와 송수신할 Slave를 선택할 때 사용되는 단자

또한, Master에서 먼저 데이터를 보내야만 Slave에서 응답할 수 있으며, Slave에서 먼저 데이터를 보낼 수 없다.

> **Register**

SPI 통신을 위해 사용되는 레지스터는 크게 3개이다.

- SPI Control Register
    - SPI 동작을 제어하기 위한 레지스터로, Interrupt enable, 동작 모드 설정, Clock 모드 설정 등에 사용
        
        
        | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
        | --- | --- | --- | --- | --- | --- | --- | --- |
        | SPIE | SPE | DORD | MSTR | CPOL | CPHA | SPR1 | SPR0 |
    - SPIE : SPI Interrupt Enable. if set to 1, enable Interrupt
    - SPE : SPI Enable. if set to 1, enable SPI communication
    - DORD : Data ORDer. if set to 1, transmit from the LSB, and if set to 0, transmit from the MSB
    - MSTR : Master/Slave Select. if set to 1 → operate Master / if set to 0 → operate Slave
    - CPOL : Clock POLarity. if set to 1, rising edge / if set to 0, falling edge
    - CPHA : Clock PHAse
    - SPR1 & SPR0 : Clock Rate select
    
- SPI Status Regstier
    - SPI 의 상태를 레지스터로, Interrupt Flag나, 클럭 주파수 설정 등에 사용
        
        
        | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
        | --- | --- | --- | --- | --- | --- | --- | --- |
        | SPIE | WCOL |  |  |  |  |  | SPI2X |
    - SPIF : SPi Interrupt Flag. 전송 완료 시, 1로 set 되면서 인터럽트가 요청 됨
    - WOCL : Write COLision flag. SPI를 통해 데이터를 전송하고 있는 동안 SDR 레지스터를 기록하려고하면 1로 set. SPSR을 읽고 SPDR에 접근하는 경우에 SPIF와 함께 클리어
    - SPI2X : SPI Double speed.

- SPI Data Register
    - SPI의 데이터 전송에 사용되는 레지스터
        
        
        | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
        | --- | --- | --- | --- | --- | --- | --- | --- |
        | SPDR7 | SPDR6 | SPDR5 | SPDR4 | SPDR3 | SPDR2 | SPDR1 | SPDR0 |
    - 레지스터 안이 데이터로 채워지고 shifting 되면서 Master ↔ Slave 간 데이터를 송수신한다.

---

참고 자료

https://electronic-king.tistory.com/16

https://m.blog.naver.com/kimwonseok77/40074376915
