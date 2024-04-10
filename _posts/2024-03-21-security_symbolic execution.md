---
title: "[보안] Symbolic Execution이란"
last_modified_at: 2024-03-21T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - security

tags:
  - security
---
## 정의

- 프로그램 실행 시 구체적인 값을 바탕으로 프로그램을 분석하는 방식
- 프로그램 실행 시 구체적인 값이 아닌 미지수가 사용된다면 해당 값을 기호값으로 가정하여 프로그램을 분석하는 방식

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/267440ba-428c-4bb7-bcd8-822ded88fc56)

## 한계

- Path Explosion
    - 대형 프로그램의 모든 실행 가능한 경로를 확인하기 어려움(효율 ▽)
    - 프로그램 크기에 따라 실행 가능한 경로의 수가 기하 급수적으로 증가 + 무한 반복문 처리 어려움
    - 해결 방법
        - 휴리스틱 하게 적용
        - 독립적인 경로를 병렬 처리(실행 시간 단축)
        - 유사한 경로 병합
- Program-Dependent Efficacy
    - 다른 테스팅 패러다임들이 사용하는 입력별 프로그램 분석보다 이점이 있는 **경로별 프로그램 분석**에 사용됨
    - 입력이 적은 프로그램의 경우, 각 입력 별로 테스트하는 것이 비용 절약
- Environment Interactions
    - 시스템 호출, 신호 수신 등을 수행하여 환경과 상호 작용
    - 제어할 수 없는 구성 요소에 도달할 때 일관성 문제 발생

## 도구

- Triton
- **angr**

## 응용: **Concolic execution**

- Concrete + Symbolic Concolioc
- 프로그램의 정확성 보다, 소프트웨어 버그를 찾기 위한 방법
- 동작 방식
    - 사용자 입력값을 저장하는 변수들은 모두 기호 변수로 처리
        - 다른 모든 변수들은 구체적인 값으로 처리
    - 임의의 입력을 선택해 프로그램을 실행
    - 실행 과정에서 Concolic 수행에 필요한 심볼들을 추출
    - 추출된 심볼을 바탕으로 symbolic 조건 생성
    - Solver를 사용해 symbolic 조건에 만족하는 새로운 입력값을 생성하여 프로그램 재실행

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/31283777-5417-4534-9352-836e0aace0f7)

<aside>
💡 해당 글은 다음 사이트를 참고하였음 https://www.lazenca.net/display/TEC/02.TechNote

</aside>
