---
title: "[보안] Dynamic Program Analysis란"
last_modified_at: 2024-03-24T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - security

tags:
  - security
---
- 정의
    - 실제 또는 가상 프로세서에서 프로그램을 실행하여 컴퓨터 소프트웨어를 분석
- 특징
    - 동적 분석을 효과적으로 수행하기 위해, 대상 프로그램이 **흥미로운 동작을 일으키기에 충분한 테스트 입력**으로 실행되어야 함
        - 코드 범위와 같은 소프트웨어 테스팅 수단을 사용하면 프로그램 동작의 적절한 부분 관찰 가능
        - 계측이 대상 프로그램의 실행(일시적인 속성 포함)에 미치는 영향을 최소화하기 위해 주의를 기울여야 함
        - 부적절한 테스트는 동적 실행 오류(런타임 오류)로 인해 치명적인 오류가 발생할 수 있음
- 계측 기법
    - 정적 바이너리 계측기(Static binary instrumentation)
        - 프로그램이 실행되기 전에 object code, executable code를 재작성 하는 단계에서 발생
    - 동적 바이너리 계측기(Dynamic binary instrumentation)
        - 런타임 단계에서 발생
        - 분석 코드는 클라이언트 프로세스에 접목된 프로그램이나 외부 프로세스에 의해 주입될 수 있음
        - 클라이언트가 동적 링크 코드를 사용하는 경우 동적 링커가 작업을 완료한 후에 분석 코드를 추가해야 함
- 테스트 종류
    - Unit Test
    - Integration Test
    - System Test
    - Acceptance Test
- 분석 코드
    - 인라인 방식으로 삽입 가능
    - 인라인 분석 코드에서 호출되는 외부 루틴 포함 가능
    - 속도 제외, 성능 측정 및 버그 식별 등의 작업하는 측면에서 수행
    - **메타 데이터라고 하는 분석 상태를 유지해야 함**
- 도구
    - ASAN - Address Sanitizer
    - Valgrind - Memcheck

<aside>
💡 해당 글은 다음 사이트를 참고하였음 https://www.lazenca.net/display/TEC/02.TechNote

</aside>
