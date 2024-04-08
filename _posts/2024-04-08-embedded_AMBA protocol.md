---
title: "[임베디드] AMBA Protocol"
last_modified_at: 2024-04-08T16:20:02-05:00
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
요즘도 드론 보안에 대한 연구를 계속 하고 있다. 드론 통신 쪽을 보면, 특히 “프로토콜(Protocol)”이라는 말을 자주하게 되는데 네트워크 및 통신 분야에서 가장 많이 쓰게 되는 것 같다. 그러나 당연히, 이 프로토콜을 네트워크에서만 쓸 수 있는 말은 아니고, 컴퓨터에서 사용하는 “데이터 교환 방식을 정의하는 규약”에 해당한다면 어디에서나 쓸 수 있다.

흔히들 드론 to 드론과 같은, 디바이스 to 디바이스 통신 프로토콜을 많이 생각하지만, 이번 포스팅에서 다룰 프로토콜은 “SoC 칩 내에서 사용하는 통신 규약”이다.

# AMBA(Advanced Microcontroller Bus Architecture)

> **정의**

SoC(System on Chip)의 구조를 보면, ARM(웬만히 언급하는 Core는 ARM Core라고 생각하면 된다) core를 기준으로 여러 디바이스들을 블록화 한 것이 한 개의 Chip에 모두 들어 있다. 이런 블록을 IP(Intellectual Property)라고 칭하며, USB, DMAN, UART 등이 있다. 그리고 이런 블록 사이에 데이터가 교환될 수 있도록하는 **Bus Protocol**이 바로 AMBA이다. AMBA는 ARM core 통신에 최적화 되어 있으며 심지어 무료다. 정리하자면, **ARM SoC chip 내부에서는 IP끼리 AMBA Protocol을 통해 통신한다**고 생각하면 된다.

> **Bus 구조**

AMBA는 3가지 종류의 Bus Interface를 사용한다.

- AHB(Advanced High Performance Bus)
- ASB(Advanced System Bus)
- APB(Advanced Peripheral Bus)

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/76a0b27c-fe4a-422d-90fb-26e28bfacf05)

위 그림은 AMBA  통신의 scheme을 보여준다. MCU 내부에 여러 IP들이 Bus로 연결되어 있다.

AHB는 High Performance인 만큼 빠른 Data 전송에 사용되며, APB는 속도가 비교적 덜 중요한 주변장치들과 연결되어 있다. 그럼 ASB는? 원래 AMBA에는 ASB와 APB가 사용되었으나, Multiplex Bus(주소/제어/데이터 라인 모두 하나로 공유하는 버스)기반의 더 좋은 AHB가 발표되면서 그 자리가 대체 되었다. 이 말인 즉, AHB 자리에 ASB가 사용된다는 것이다.

AHB로 연결된 구역과 APB로 연결된 구역 사이에 “Bridge”가 보이는데, 이는 AHB와 APB의 속도와 규약이 다르다 보니 이를 연결해주는 장치이다. 그럼 왜 굳이 Bridge라는 장치를 쓰면서 AHB회로와 APB 회로를 나눌까? 그건 장치마다 속도가 다르기 때문에 떨어지는 효율 문제를 해결하기 위해서다. 

AHB로 연결된 디바이스들은 APB로 연결된 주변 장치들보다 속도가 매우 빠르기 때문에, 만약 두 회로를 구분해 놓지 않고 모두 AHB로 연결해둔다면 빠른 디바이스가 중간에 느린 디바이스에게 데이터를 요청하고 응답을 받을 때까지 Bus를 사용하지 못하고 놀게 된다. 이런 문제를 해결하기 위해 빠른 디바이스끼리 묶어놓고, 느린 디바이스에게 요청을 할 때는 Bridge에 요청을 던져두게 하는 것이다. 

> **Master & Slave 구조**

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/4addd808-1be1-4c65-bd64-d88be4372e91)

 위에서 Bus 중심 scheme으로 보았다면, 이번엔 IP 중심으로 확인해보자. 생긴 게 어딘가 익숙하지 않은가? 맞다. 비슷한 Master ↔ Slave 구조를 지난 번 포스팅인 SPI 통신에서도 확인했었다. 정말 많은, 웬만한 디바이스 컨트롤은 이와 유사하게 생겼다.

Master와 Slave는 최대 16개까지 사용할 수 있다.

 아무튼! 위 그림을 보면 Master와 Slave 사이에 “Arbiter”와 “Decoder”가 끼어있는 것을 볼 수 있는데, Arbiter는 Bus를 컨트롤 하는 역할을, Decoder는 Master와 Slave의 중간에서 중재자의 역학을 한다. AMBA의 신호 입력 순서를 보면 더 이해하기 쉽다.

> **AMBA 신호 입력 순서**

(참고: x는 master 또는 slave 번호)

1. Master → Arbiter : **HBUSREQx**
    - Master가 Bus를 사용하기 전 Arbiter에게 허락을 구함
2. Arbiter → Master : **HGRANTx**
    - Bus를 사용해도 된다면, Master에게 가능하다는 신호를 보냄
3. Master → Arbiter : **HLOCK**
    - Bus를 사용하기 전, 사용한다는 사실을 Arbiter에게 알림. Arbiter는 현재 버스를 사용하고 있는 Master에 대한 정보를 기록해놓고 다른 Master가 또 Bus를 사용하고 싶어하면 Priority에 맞게 허가를 함
4. Master → Decoder : **HADDR**
    - Master가 접근하고자 하는 Slave의 주소를 Decoder에게 보냄
5. Decoder → Slave : **HSELx**
    - Slave 동작 활성화
6. Master → Slave : **HWRITE**
    - Write하겠다는 신호로, High로 전송 (Low면 read)
7. Slave → Master : **HREADY**
    - Slave 장치가 준비가 완료되면, 준비가 완료 됐다는 신호 전송
    - Slave 장치는 Master 장치보다 느린 경우가 많으므로, HREADY가 low 신호인 경우 Master가 Slave에 접근하지 않음
8. Master → Slave : **HWDATA**
    - 데이터 전송





참고 자료

책, 저자 히언, <임베디드 레시피>

https://recipes.tistory.com/312