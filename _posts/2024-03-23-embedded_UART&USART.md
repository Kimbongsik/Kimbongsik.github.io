---
title: "[임베디드] UART & USART 통신"
last_modified_at: 2024-03-23T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
임베디드 개발할 때 참 많이 쓰는 것 중 하나가 시리얼 통신, UART 통신이다.

간단하게 글로 정리해본다.

## UART 통신이란

- UART(Unicersal Asynchronous Receiver / Transmitter)
- 병렬 데이터를 직렬 방식으로 전환하여 데이터를 전송하는 컴퓨터 하드웨어

흔히 통신 방식에는 직렬 방식과 병렬 방식이 있다. 직렬(Serial) 통신은 한 개의 입출력핀을 통해 n개의 비트를 한 번에 전송하고, 병렬(Parallel) 통신은 n개의 비트를 전송하기 위해 n개의 입출력핀을 사용한다. 아두이노 등, 많은 임베디드 보드에서 제공하는 방식은 대개 시리얼 통신 방식이며, 그중 가장 많이 쓰이는 방식 중 하나가 UART이다. 

## UART 특징

### 비동기 통신

- **UART는 비동기 통신**을 사용한다. 동기 통신 방식과의 차이점은, 클럭이 존재하지 않아 데이터의 시작과 끝을 구별하기 위해 특정 비트로 표시를 해둔다는 거다. 반면에 동기 통신에는 클럭이 존재하여 이에 의해 비트를 구별하게 되므로, 별도의 비트를 표시하지는 않고 프레임 단위로 데이터를 보낸다.
- 다음은 UART의 데이터 형태를 나타낸다. start bit와 stop bit가 있는 것을 확인할 수 있다.

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/92cc25c1-7cd8-4f67-952f-18982a9696ca)

### Baud rate

- UART 전송을 사용하기 위해서는 수신측과 송신측이 서로 연결되어 있어야 하는데, 수신측을 RX(Recieve), 송신측을 TX(Transmit)라고 한다.
- 시리얼 통신을 사용하기 위해서는 TX / RX 간에 데이터를 주고받는 속도에 대한 규약이 필요한데, 이를 보율(Baud rate)로 정했다.

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/fc14400c-a1de-4b74-9efb-1843a7009314)

![3](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/75e9e1c8-c034-4caa-a256-8863721c1a9f)

## USART 특징

### 동기 통신

- **USART는 동기 모드**에 해당하는 UART 시리얼 통신이다. UART와 달리 Half duplex(송수신 모두 가능하나, 한 번의 순간에 둘 중 하나만 보낼 수 있다. UART는 Full duplex로, 송수신 데이터를 동시에 전송할 수 있다.)
- 동기 통신 방식으로 인해, USART를 사용하기 위해선 Clock을 위한 회선이 하나 더 필요하다.
- 송신 측에서 이진 데이터들을 정상적인 속도로 보내면, 수신 측에서는 클럭 사이클 간격으로 데이터를 인식하여 읽는다.
- 동기식 전송은 데이터를 구분할 필요 없이 클락 유무만 확인하면 되기 때문에 데이터 송수신 효율이 높다.

![4](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/0febdede-a4e0-4b41-9131-2e5800507c42)
