---
title: "[임베디드] 임베디드 리눅스(Embedded linux)란"
last_modified_at: 2023-04-01T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
<br/>

## Kernel vs Application Program

- 수행 시기

-Application Program(순차적으로 수행)

-Kernel : system call, interrupt, boot

## RTOS vs Full function OS

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2eb170a7-db96-4684-ae79-7bb33eb6872b/Untitled.png)

커널의 크기는 RTOS는 작고, Full function OS는 크다. 커널 오버헤드는 그 반대.

Full functin OS에서 real time features는 고려되지 않지만 CPU의 속도가 빨라 임베디드 시스템에서도 사용되고 있음. 그러나 hard real time system에서는 RTOS가 사용됨 .

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/709b841b-2e4a-43e7-a32e-5b0d5c226a1a/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/632df2b1-f197-48ae-aef0-1cd3961745f4/Untitled.png)

임베디드는 특정 역할만 수행하면 되기 때문에 virtual memory 기술이 필요는 없지만 이로 인한 memory protection은 필요함. 

예시)

RTOS를 이용하여 multi-task로 여러명이 각 task를 개발하여 통합하는 경우

→ memory protection을 제공하지 않음. 특정 task의 포인터 버그로 다른 task를 죽일 수 있음. 

vs

리눅스 상에서 멀티 프로세스에서 각자 개발하여 통합하는 경우

→memory protection 제공. 특정 포인터 버그는 자기의 프로세스를 죽임. (훨씬 유리)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/47322b1b-d851-445f-9593-91704fa02ac3/Untitled.png)

아두이노에서 I/O 제어는 우리가 직접했음. 그러나 full-function os는 os에서만 제어가 가능함. 

## What is embdded Linux

일반 linux와 커널이 동일함. file system에서는 차이가 좀 남. 왜 embedded linux를 쓸까?

1. 공짜임
2. Re-using components, full control, quality, easy testing of new feautres, community support, taking part into the community

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/15bbe4ba-2e6e-4b7e-a388-ac0a1cee6a88/Untitled.png)

- cross development environment (교차 개발 환경)

→Host라고 불리는 개발 PC에서 개발이 이뤄지고, Target이라고 불리는 Embedded system에서 실행됨. 

- compiler, debugger 등을 “tool chain”이라고 하며 host PC에 구축을 해야 함.
- Bootloader, linux kernel, library들은 일반적인 구성요소이므로 이를 개발하여 제공하는 업체와 이를 이용하여 application을 만드는 업체는 일반적으로 구분되어 있음.

## Cross develpment environment(교차 개발 환경)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4442d60d-ce5d-4de9-b056-98b570ab1d27/Untitled.png)

- serial, ethernet을 통해 연결
- 하드웨어 디버깅을 위해 JTAG이 사용되는데, bootloader 다운로드도 JTAG을 사용함. 커널 및 어플리케이션 다운로드도 JTAG을 이용하거나 bootloader에서 제공할 수도 있음. 또한 별도의 펌웨어 업데이트 방법을 제공할 수도 있음.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b04a8a75-0069-4bea-a0fb-7830ead5382f/Untitled.png)

- compilatin machine : HOST
- execution machine : target
- Cross compile ↔ Native compile
