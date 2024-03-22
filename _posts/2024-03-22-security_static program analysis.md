---
title: "[보안] Static Program Analysis란"
last_modified_at: 2024-03-22T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - cs
  - security

tags:
  - cs
  - security
---
- 정의
    - 프로그램을 실행하지 않고 프로그램 분석
- 분석 대상
    - 소스코드
    - 오브젝트 코드
- 분석 방법: 구문 검사, 의미추론 등
    - Unit level
        - 프로그램 문맥과의 연결 없이 특정한 프로그램 안이나 서브루틴에서 발생하는 분석
    - Technology Level
        - 문제를 찾고 긍정 오류를 피하기 위해서 & 프로그램의 전체적이고 의미적인 관점을 얻기 위해 유닛 프로그램들 간의 상호작용을 고려하는 분석
    - System Level
        - 유닛 프로그램들 간의 상호작용들은 고려하지만 한 기술이나 한 언어에 제한되지 않는 분석
- 사용 기술
    - Abstract Interpretation
    - Data-Flow analysis
    - Model Checking
    - Symbolic execution
- 분석 내용
    - 보안 취약성
    - 코딩 표준 위반
    - 사용되지 않는 변수 & 사용되지 않는 코드
    - 정의 되지 않은 값의 변수 접근
    - 코드와 소프트웨어 모델의 구문 규칙 위반
    - 모듈과 컴포넌트 간에 일관되지 않은 인터페이스

- 도구
    - Clang Static Analyzer

<aside>
💡 해당 글은 다음 사이트를 참고하였음 https://www.lazenca.net/display/TEC/02.TechNote

</aside>