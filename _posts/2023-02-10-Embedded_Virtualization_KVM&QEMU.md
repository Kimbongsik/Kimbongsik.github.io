---
title: "[가상화] KVM, QEMU"
last_modified_at: 2023-02-10T16:20:02-05:00
categories:
  - embedded

tags:
  - embedded
---


<aside>
💡 해당 포스팅은 isnt님의 블로그(링크: [https://m.blog.naver.com/ilikebigmac/222009981745])의 포스트를 보고 정리한 내용입니다.
</aside>  
  
  
  
# KVM

### KVM의 하이퍼바이저

- 리눅스 커널과 직접 통합된 하이퍼바이저로, 커널의 기능 중 하나로 지원되고 있다. KVM 또한 Full Virtualization임
- Baremetal hypervisor → Thin hypervisor
    - Baremetal(Native, Type1)형 하이퍼바이저
    - ssh를 제외한 어떠한 서비스도 제공되어서는 안 되며, 오로지 가상화 관리를 위해서만 사용
    - 스토리지 가상화에 필요한 개념으로, 생성 시에는 필요한 최소한의 공간만을 할당하고 추후 사용량에 따라 동적을 공간을 늘리는 방식
- Hosted hypervisor → Thick Hypervisor
    - Hosted(Type 2)형 하이퍼바이저
    - 일반 운영체제 위에 하이퍼바이저 소프트웨어를 설치하여 사용
    - 게스트 OS를 올리기 쉽기 때문에 테스트, 시연용 등으로 많이 사용
    - 생성 시에 필요한 공간과 성능이 고정됨

### VirtIO

- 전가상화에서 VM은 자신이 가상이라는 사실을 인지하지 못함
- 위 사실 때문에, CPU나 메모리는 드라이버가 따로 없어 I/O 장치들은 반 가상화에 비해 중간 과정이 많음
- 이로 인해 전가상화에서 오버헤드가 발생
- 그래서 디스크나 네트워크 카드 같은 입출력 장치에 대해 반 가상화를 도입 → VirtIO 타입
- VirtIO: Para Virtualized Driver
    - 가상의 I/O 레이턴시를 감소시키고 호스트와 비슷한 수준까지 thoroughput(처리율)을 증가 시킴
    - 따라서 KVM에서 장치를 붙일 대 기왕이면 VirtIO 타입으로 붙이는 것이 좋음



# QEMU

- KVM은 CPU와 램 가상화
- QEMU는 각종 디바이스 가상화
- 즉, QEMU는 애뮬레이터임(소프트웨어적으로 동작 시킨다는 의미)
- Guest CPU의 명령을 실행 도중에 Host CPU의 명령으로 동적 번역 해주는 역할을 함
- 이식성이 매우 높기 때문에 모든 하드웨어 장치를 애뮬레이트 할 수 있음
    - 해당 장점 때문에 KVM과 함께 매우 자주 사용된다고 함