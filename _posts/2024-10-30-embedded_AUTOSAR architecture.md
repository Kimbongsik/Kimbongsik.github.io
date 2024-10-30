---
title: "[임베디드] AUTOSAR란"
last_modified_at: 2024-10-30T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
요즘 자동차는 굴러가는 거대한 컴퓨터라고 해도 과언이 아닌 것 같다. 기계 제어에 의존도가 높던 과거와 달리, 최근 생산되는 자동차는 전자 제어기(Electronic Control Unit, 이하 ECU)의 사용 비중이 매우 높아지고 있다. 차량을 제작하는 회사와 ECU를 제작하는 회사는 나누어져 있고, 따라서 이 둘을 결합하기 위한 부품의 표준화가 필요하다. 이때 ECU는 전자 장치이기 때문에 하드웨어 뿐 아니라 올라가는 소프트웨어에 대한 표준화 작업 또한 필요한데, 이 **AUTOSAR가 자동차 ECU의 표준 소프트웨어 아키텍처이다.**   

# AUTOSAR(AUTomotive Open System ARchitecture)란

- 약자의 의미 그대로, 자동차의 개방형 시스템 구조임
- 자동차 부품을 제어하기 위한 SW의 표준화를 목표로, 여러 자동차 관련 업계에서 함께 참여하고 있는 세계적인 개발 파트너십임
- BMW, Ford, Toyta, Volvo, Volkswagen, 현대 등 대부분의 자동차 제조업체에서 AUTOSAR 표준을 사용하고 있음  

# AUTOSAR 구조

- AUTOSAR는 크게 3가지 계층으로 나누어짐

![1](https://github.com/user-attachments/assets/43383c3a-305c-44b9-885b-d9054196e2eb)

### AUTOSAR Software

ECU로 동작하고자 하는 기능들을 구현하는 계층. 여러 Component 들(Software Component, 이하 SWC)로 구성되어 있음. 독립된 실행 단위.  

![2](https://github.com/user-attachments/assets/603099e9-6102-4125-a087-3fc98b21802f)
### RTE(Runtime Environment)

BSW와 AUTOSAR Software 사이 데이터 이동 통로 역할. 일종의 추상&가상 환경을 통해 소프트웨어가 하드웨어와 독립적으로 동작할 수 있도록 하는 계층  

- 기능: AUTOSAR Software 구성요소를 특정 ECU에 대한 매핑과 독립적으로 만듦
- 구성
    - AUTOSAR 구성 요소 및 AUTOSAR 센서, 액츄에이터 부품이 포함된 어플리케이션에 대한 통신 서비스를 제공하는 미들웨어

![3](https://github.com/user-attachments/assets/f750fac2-491f-40c4-92bf-9e3e9c147190)  

### BSW(Basic Software)

여러 ECU에서 공통으로 사용하는 기능들이 구현되어 있는 계층. 구성 요소는 아래와 같다.  

- Serivces Layer
    - 기능: 가장 높은 베이직 소프트웨어 계층. 어플리케이션에 대한 기본 서비스 및 기본 소프트웨어 모듈 제공
    - 구성
        - OS
        - Network communications and management
        - NVRAM management
        - 진단 서비스(UDS 프로토콜 토함)
        - ECU 상태 관리
- ECU Abstraction Layer
    - 기능: ECU 하드웨어 레이아웃과 독립적으로 상위 계층 소프트웨어를 만들 수 있도록 구축해놓은 추상 계층
    - 구성
        - 주변 장치 접근
        - MCU 인터페이스 API(포트 핀 등)
- Complex Drivers
    - 기능: 센서 및 엑츄에이터 작동을 위한 특수 기능을 관리하는 드라이버
    - 구성
        - 전기 값 제어, 위치 증가 감지 등.. (MCU에 직접 액세스)
- Microcontroller Abstraction Layer(MCAL)
    - 기능: MCU와 독립적으로 상위 계층의 소프트웨어를 만들 수 있도록 구축해둔 추상 계층
    - 구성
        - On-chip MCU 주변 장치 모듈 및 외부 디바이스에 직접 접근하는 소프트웨어 모듈

![4](https://github.com/user-attachments/assets/73a45205-288d-4e6d-9086-95de06646570)  
  





-참고 자료: https://www.renesas.com/en/products/automotive-products/autosar/autosar-layered-architecture