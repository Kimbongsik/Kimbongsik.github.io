---
title: "[임베디드] MCU 동작과정"
last_modified_at: 2023-03-19T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
<br/>


일반적으로 윈도우 PC를 통해 MCU 프로그래밍을 수행하며, 작업이 완료된 프로그램을 MCU에 넣기 위해 디버거 및 프로그래머가 필요하다.

- 예를 들어, ST마이크로일레트로닉스 제품인 ST-LINK 디버거가 있다. PCB(MCU를 사용하기 위해 필요함)와 ST-LINK는 전원 공급 모니터를 위한 2개 라인, 1개의 리셋 라인, 1개의 통제 신호라인으로 연결되어 있다.

# MCU 내부 구조

간략한 내부 구조는 다음과 같다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/00f8177e-f150-4c43-89a6-22177600b978/Untitled.png)

- AD/DA 전환 모듈
    - 아날로그 데이터 → 디지털 데이터: ADC
    - 디지털 데이터 → 아날로그 데이터: DAC
    - 아날로그 데이터를 비교하기 위한 컴퍼레이터
- 타이머
    - 입력 캡쳐, 토글, PWM(Pulse Width Modulation: 펄스 폭 변조) 등,,

# MCU 동작 구조

1. 프로그램을 ROM에 입력(이 때, PC 사용)
2. MCU가 켜지면 CPU가 활성화되어 ROM으로부터 프로그램 read
3. CPU에서 데이터 연산 및 처리(이 때, RAM과 레지스터를 임시 저장용으로 사용)
4. 기능 수행
5. CPU가 보조 모듈에 동작 명령을 전송
6. 보조 모듈과 외부 회로 간 교환된 데이터가 CPU와 교환
    - CPU ↔ 보조 묘듈 ↔ 외부 회로
