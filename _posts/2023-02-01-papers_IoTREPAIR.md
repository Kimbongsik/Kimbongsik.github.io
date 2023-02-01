---
title: "IoTREPAIR: Systematically Addressing Device Faults in Commodity IoT"
last_modified_at: 2023-01-16T16:20:02-05:00
categories:
  - papers

tags:
  - Usenix
  - embedded
  - 임베디드
  - 보안 
---


# Abstract

- IoT 디바이스들은 분권화 되고 불안정한 환경에서 실행되기 때문에 power failure나 network disruption과 같은 다양한 종류의 Fault를 야기했음
- 그러나 현재 IoT 플랫폼들은 프로그래머들이 compexity와 error-prone(발생하기 쉬운 오류)에 대한 fault들을 수동으로 다루기를 요구함

>제안

- IoTREPAIR: fault handling system for IoT
    - fault 식별 모듈과 통합된 fault handling system
    - 효과적으로 여러 fault type들을 처리하는 fault handling 라이브러리 제공
    - IoT fault handling에 주도권이 있는 상위 라이브러리(디바이스, 유저 선호도, 개발자 설정을 입력으로 하는)

>결과

- IoTREPAIR를 통해 unsafe & insecure한 디바이스에서 평균 63.51%의 error fault를 감소 시킴

>결론

- IoT fault handler에 대한 시스템적 디자인을 통해  유저가 유연하고 편리하게 복잡한 UiT fault handling을 수행할 수 있도록 함

# Introduction

- 사물인터넷은 점점 많이 사용되고 있으며 많은 기기가 생겨 여러 IoT 디바이스들과 IoT 환경들이 생겨남.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/47b8880f-b3cd-49b3-9d71-6d3d300c6862/Untitled.png)

- IoT 디바이스는 대개 아래 구조를 포함함
    - 위치, 온도, 광학 센서 등의 센서
    - raw 센서값을 읽을 보조 MCU
    - 저전력 CPU 코어
    - 네트워크 인터페이스
    - 배터리 및 파워 서플라이 유닛
- 해당 앱은 온도 측정을 위해 온도 센서를 이용하고, 이를 통해 창문을 여닫고 히터를 조정하는 등의 일을 수행하는 등, IoT 앱은 주변 환경과 interaction을 수행함
- IoT 기기는 실제 배포를 위해 높은 신뢰성을 유지해야 하지만, 아래의 원인으로 인해 여러 faults가 발생함
    - power outages
    - network distruption
    - IoT 디바이스가 low computation 환경이라는 제약이 있기 때문에 더욱 자주 발생함
    - small batteries
    - 날씨 및 충돌과 같은 환경적 조건 같은 파괴적인 문제
    - user error
    - hw, sw의 결함(flaws)
- 한 연구에 따르면 스마트 홈의 장치는 정전, 네트워크 중단 및 하드웨어 문제로 인해 하루에  4시간 이상의 오류가 발생할 수 있으며 최근 연구에서는 온도 센서의 온도 판독 오류가 15% 이상 발생하는 것으로 나타남
- Faults를 세 가지로 분류함
    - Fail-stop
        - 디바이스가 기능을 멈춰 외부 요청에 무응답
        - ex: 디바이스 전력 로스로 인해 센서 판독 및 작동이 중단 되는 경우
    - Non fail-stop
        - 디바이스가 올바르게 수행하지 않고 다르게 응답하는 경우
        - 올바르지 않은 센서 읽기, 요구된 지시에 적절하게 응답하지 않는 경우
        - ex: 앱으로부터 올바르지 않은 상태를 빈번하게 유발할 수 있는 상태를 반복하는 소프트웨어 에러
    - Cascading faults(계단식 오류)
        - 장치가 다른 앱에서 이벤트를 잘못 트리거하여 초기 오류가 시스템을 통해 계단식으로 발생하는 경우
        - ex: 히터를 키는 경우, 다른 에너지 절약 앱이 에어컨을 끔

# Overview

- IoTREPAIR 아키텍쳐

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08ab421d-d047-4372-a83d-0cf0a40270c7/Untitled.png)

- configuration file을 포함(1)
    - iintialization phase 동안 생성
    - 이후 IoTREPAIR, 사용자, 개발자들에 의해 커스터마이징 가능
    - IoTREPAIR의 기능을 관리 및 컨트롤할 파라미터 셋을 포함
        - ex: application suppression이 활성화 되었는지 여부와 결함 식별 모듈 식별 시간의 상한선 정의
    - → 다양한 faults에 대해 유연하게 처리 가능
- faulty 디바이스를 추적하는 faulty 식별 모듈과 통합(2)
    - 많은 fault 식별 기술들이 존재하지만, iotrepair은 오직 fault handling 자동화를 위해 식별된 결함에 따라 부여된 디바이스 ID만을 요구한다.(3)
    - restart, retry, checkpoint 같은 기능과 같이, 여러 fault type을 완화 및 수리하기 위한 fault handling library를 포함(4)
        - 각 기능은 fault hadling을 시도하는 동안 configuration file에 있는 파라미터를 사용
    - 개발자는 fault handling logic을 configuration file을 수정하기 위해 제공된 API를 사용하거나 try/catch문을 앱 소스 코드에 삽입하는 방식으로 향상 시킬 수 있음(5)
- configuration 파일, fault handling functions, 개발자의 노력을 최소화 히키는 스키마를 통한 fault handling 자동화(6)
    - 스키마: 특정 순서로 결함 처리 기능 집합을 호출하는 기능 목록으로 구성됨(7)
        - 결함이 있는 장치가 식별되면 iotrepair는 결함이 있는 장치에 대한 폴링 및 결함 디바이스에 전송되는 명령들을 차단하는 장치 억제를 수행
        - 그 다음 configuration file에서 장치에 대해 정의된 스키마 실행

## Design Requirements

- requires the ability to query list of connected devices, poll devices states, query installed apps, send actuation commands, create and manage a configuration file, expose library functions to apps, and hoot fault identification messages
- installation space must be between the apps and devicees
    - events and actuations can be interrupted
- requires the ability to query types and capabilities at initialization to create configuration file

## Configuration File

- three types of information
    1. general parameters
        - defing the upper bound on how long fault identification module takes to identify a fault as well as specifications for checkpoint cleanup and replicated(복제된) device detection, each specified by the user
    2. device-based parameters
        - defining the list of devices running in the system and the parameters (ex: which fault handling scheme) for each device
        - default configurations are generated for each device connected to the edge device by selecting from a list of default configurations that best matches the device type
            - can be modified by the user, by the automated handler at runtime, or by developers by making API calls in their apps
    3. app parameters
        - listing what apps are installed on the edge device and whether the application suppression is evabled for each;
            
            -by default application suppression is disabled
            

# Fault-handling functions

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b790d61c-761a-476d-be2f-d2aff838831f/Untitled.png)

- introduce a set of functions to handle faults in three groups(table 2)

## Device-based Functions

- Device-based functions implement *isolated fault handling*, which does not consider the overall state of the system(i.e., state of all connected devices), and acts based only on the state and configuration of the faulty device
- **Activate Redundant Device**
    - define replicating devices to be of two types
        1. a duplicate of the primary device
        2. a devices that provides similar capabilities(ex: a camera that can detect motion to replicate a motion sensor)
        
        → IoTREPAIR currently only supports type a replication
        
    - **allows the system to continue to run** unaffected by a detected fault, so long as the configuration file specifies the replicating device ID if one exists
        - if exist, **it redirects polls and actuation commands from the faulty device to the replicating device**
    - using this method, iotrepair does not rely on native replication support from the platform
    - iotrepair **examines the list of connected devices** through platform capabilities for devices with matching type, and **automatically generates relevant configuration data**
        - replicate device location is not necessarily co-located
        - to address this, observes the sensor states during polling for a configured time and checks whether devices report the same states consistently
        - if the two devices’ states and transition timings remain within acceptable bounds, these devices are identified to be replicating
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e7eed2d-90d2-44d8-818f-ad804dd9491a/Untitled.png)
        
        - Fig3: how wo colocated motion sensors would be detected as replicating, as their states match consistently, while an unrelated presence sensor and distant motion sensor are discarded
        
- **Retry Device State**
    - Retry aims to prevent overhandling short
    - 재시도는 결함이 해결되거나 구성 파일에서 이 장치에 대한 재시도 제한에 도달할 때까지 결함이 있는 장치의 상태를 지속적으로 폴링
        - 예를 들어 선택적으로 결함이 있는 장치가 액추에이터이고 원하는 상태가 알려진 경우 임시 중단으로 인한 결함을 해결할 수 있는 폴링 사이에 교정 작동을 시도할 수 있음
- **Software and Hardware Restart**
    - 소프트웨어 버그나 하드웨어 오류 수정 시 유용
    - SoftwareRestrat 및 HardwareRestart 기능은 재시작을 위해 장치에 신호를 보냄
        - 많은 IoT 장치와 아두이노 같은 프랫폼은 재시작을 구현해뒀음
        - iotrepair는 응답 신호를 기다리고 장치 상태를 모니터링하며 재시작이 완료되지 않으면 구성에 따라 여러 번 재시도 한 다음, 기능의 오류가 해결되었는지 확인

## Environment-based Functions

- 연쇄 오류(cascading error)를 완화하기 위해 연결된(linked) 오류처리를 구현
- **Checkpoint**
    - 체크포인트는 호출 당시 보류 중인 대기열에 모든 장치 상태 모음을 저장
    - The automated fault handler invokes the checkpoint function after every actuation where no subscribing application initiates an actuation based on the new state
    - 체크포인트가 유효한 것으로 간주되고 기록 로그에 추가되기 전에 약간의 시간이 지나야 합니다. 지연 기간은 결함 식별 모듈에서 제공하거나 기본값으로 설정해야 하는 결함 감지 시간의 상한값에 의해 결정됩니다
    - 제한 사항은 작동을 트리거하는 체크포인트 또는 결함이 존재하는 위치를 생성하는 것을 방지합니다
    - 장치 상태를 센서 상태와 액츄에이터 상태로 나눔
        - sensor states
            - 읽기 전용 상태
        - actuator states
            - actuation을 통해 읽고 쓸 수 있는 상태: roll back 가능
    - IoTREPAIR에는 체크포인트 동안 장치 상태의 기록을 기록하고 롤백하는 동안 현재 센서 상태에 따라 기록을 조회하여 가장 바람직한 액츄에이터 상태를 복원하는 새로운 기록 기반 체크포인트/롤백 메커니즘이 있음
    - 유효한 체크포인트는 체크포인트와 해당 빈도를 포함하는 기록 로그에 저장됨
    - 빈도는 현재 물리적 환경과 일치할 가능성이 가장 높은 체크포인트로 롤백하는 기능을 구현할 때 필수적임
    - 주어진 액추에이터 상태 세트에 대한 가장 최근 체크포인트만 로그에 저장됨
        - 센서 상태 집합에 대한 가장 최근의 센서 상태만 저장하면 로그 크기가 폭발적으로 커지는 것을 방지할 수 있음
        - → 체크포인트는 구성된 기간 동안 사용되지 않은 상태로 남아 있으면 기록에서 제거됨
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5aa2891-9ecf-429a-81d2-1dda146dcf05/Untitled.png)
        

→ 그림4: 액츄에이터 상태 변화로 인해 시스템에서 시간이 지남에 따라 체크포인트가 어떻게 사용되는지에 대한 예시

- **Rollback**
    - 시스템 상태를 체크포인트와 일치시키기 위해 일련의 작동 수행
    - 롤백 대상으로 삼을 최상의 체크포인트를 선택하기 위한 세 가지 기술 설계
        - Most Recent
            - 최신 타임 스탬프가 있는 체크포인트
        - Fail Safe
            - 모든 액츄에이터가 설정한 fail safe 상태일 때에 있는 체크포인트만 고려
            - 이 축소된 목록에서 센서 상태가 시스템의 현재 센서 상태와 일치하는 체크포인트가 선택
            - 결함이 있는 센서의 상태를 신뢰할 수 없으므로 체크포인트가 현제 센서 상태와 일치하는지 판단할 때 해당 센서 상태를 고려할 수 없음
            - 이로인해 두 개 이상의 일치 상태가 발생하면 빈도가 tiebreaker로 사용됨
        - Fail norm
            - faile safe와 동일한 방식으로 체크포인트 일치를 찾지만 먼저 faile safe 구성을 기반으로 리스트를 줄이지는 않음
    - 롤백은 잘못된 시스템 상태로 들어갈 수 있기 때문에 위험함
        - 변경해야 하는 액츄에이터가 현재 결함이 있는 것으로 알려진 경우 롤백이 중단됨
        - 롤백이 완료되면 장치가 수리되거나 다른 롤백이 발생할 때까지 잘못된 센서 값이 체크포인트의 해당값으로 설정됨
