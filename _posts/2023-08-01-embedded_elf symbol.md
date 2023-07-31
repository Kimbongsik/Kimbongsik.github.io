---
title: "[임베디드] ELF Symbol"
last_modified_at: 2023-08-01T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---
<br/>
최근 포스팅 주기가 좀 길었다. 과제를 끝내고 논문, 보고서, 발표, 개발 등.. 7월 한 달 동안 해야 할 일이 좀 몰려 있었다.

아무튼, 이번 포스팅을 기점으로 슬슬 임베디드 및 보안에 대한 포스팅을 위주로 써보려고 한다. 

그 첫 포스팅 주제는 ELF 파일. 

# ELF 파일이란?

Executable Linkable File(ELF)로, 실행 및 링킹 가능한 파일이라는 뜻이다. Unix 계열 시스템들의 표준 바이너리 파일 형식으로, 임베디드 종사자들이라면 자주 보게 될 수밖에 없는 파일 형식이다. 일반적인 hex 파일 등과 다르게 파일 내부에 디버깅 정보, 함수 심볼 등에 대한 정보가 남아 있어 구조 분석이나 파싱에 용이하다.

 

임베디드 디바이스들은 보통 OS가 없는 베어메탈 환경에서 주로 실행되는데, 당연히 OS에 종속적인 ELF 파일 그 자체가 펌웨어로 실행되는 것은 아니다. 임베디드 개발 시 보통 크로스 컴파일을 하게 되는데, 이때 주로 사용되는 host OS가 linux이고, ARM Core에서 실행 가능한 코드 생성을 위해 (저전력, RISC 등의 특징 때문에 임베디드 디바이스에 탑재되는 상당수의 CPU(MCU)가 ARM Core를 사용한다.) arm-none-eabi-gcc 등의 툴체인을 사용해 개발하게 된다. 그러니 당연히 linux 환경에서 컴파일 했을 때 생성되는 ELF 파일을 잘 알아두는 게 좋겠죠!

# Symbol

그럼 ELF 파일 안에 있다는 ‘Symbol’이 대체 뭔지 알아보기로 하자. Symbol은 링커가 알아볼 수 있는 단위로, 링킹 후에는 자신만의 주소를 갖게되는 단위를 일컫는다. 전역변수의 이름이나 함수 이름 등이 이에 해당하고.. ELF 파일 안에서는 Symbol Table을 두어 이들을 관리한다. 

만약 다른 파일에 정의된 심볼을 가져다 쓰는 경우에는 그 해당 파일에 심볼이 없기 때문에 이 Object(링킹 전 .o 목적 파일을 생각하면 될듯)에서 Symbol table은 완전하지 못하다. 이들을 Linker에 의해 처리해서 다른 파일에 있는 심볼을 연결해 사용할 수 있게 한다. 

이 심볼은 Linker만이 사용하고, 링커는 해당 심볼들을 주소로 변환하여 바이너리로 만든다.

- Symbol
    - .text (함수, 실행 코드 등) → RO(Read Only)
    - .constdata (const 전역 변수) → RO
    - .bss (초기화 되지 않은 전역 변수) → ZI(Zero Initialized)
    - .data (초기화 된 전역 변수) → RW(Read-Write)
- Non Symbol
    - 그 외 local 변수

크게 중요한 것들은 위의 것들이다.

임베디드 디바이스에서는 Flash를 ROM이라고 생각하면 된다. 즉 전원이 꺼져도 그 내용을 계속 유지하는 메모리로, RAM에 올라가지 않고 Flash에 접근하여 가져오는 데이터이다. 예를 들어 code(.text 섹션)와 const data의 경우 값이 변하지 않고 유지되어야 하기 때문에 접근 권한은 RO(Read Only)이며, Flash에 위치하면 된다. RW 데이터의 경우 수정 가능해야 하기 때문에 Flash에도 위치해야 하며 해당 값이 RAM에도 위치해야 한다. ZI는 어차피 모든 값이 0이므로 Flash에 저장되지 않아도 되며(실제로 .bss는 파일에서 용량을 차지하지 않는다.) RAM에 위치하면 된다. 

위 Symbol과 Non Symbol의 차이는, 절대 주소를 갖느냐 갖지 않느냐의 차이이다. 예를 들어 Stack에 저장되는 지역 변수와 Heap에 저장되는 동적 할당된 변수가 이에 해당된다. 임시적으로 사용하는 데이터들이고 삭제, 변경 등을 반복하는 데이터들이기 때문에 절대 주소를 가질 필요가 없다(메모리 낭비).

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/a2782d2a-8227-491b-9e77-8487975c6fda)

왼쪽은 링킹 전의 파일 구조, 즉 Linking View고, 오른쪽이 Execution View로 링크가 끝난 후에 완전한 실행 가능한 형태이다.  ELF Header에는 엔디안, 운영체제, 무엇보다 섹션의 시작 위치(파일의 offset으로, Flash 메모리 주소로 보면 될듯) 등이 저장되어 있다. 

# Section

Section은 프로그램을 실행 시키기 쉽도록 데이터 유형에 따라 분류해 놓은 것이다. Segment는 Section의 집합인데, 앞서 언급했듯 링커가 여러 오브젝트 파일(.o)을 링킹하게 되면, 각각의 오브젝트 파일들이 가지는 .text, .bss, .data를 모두 새로운 text, bss, data에 모은다. 이것들이 Segment라고 보면 된다. 

아무튼 좀 중요한 Section 정보들이 있는데, 잠깐 다뤄보자.

## .init

- 프로그램 진입 포인트로, main 함수(실제 프로그램 실행) 실행 이전에 먼저 실행되는 코드가 있는 섹션
- 함수 __init()을 위한 명령어가 위치한 섹션

## .text

- 기계어로 변환된 프로그램 코드. PC(Program Counter)는 +word size만큼 움직이면서 fetch 할 명령어를 저장한다.

## .rodata

- Read Only 데이터를 의미하며, const, printf, switch case문에 의한 jump table 등이 포함된다.

## .data

- 초기화 된 전역 변수 및 static 변수

## .bss

- 초기화 되지 않은 전역 변수 및 static 변수
- 파일에서 아무 용량을 차지하고 있지 않고 bss 섹션 길이 정보만이 파일에 저장된다. 프로그램이 로드될 때 bss 섹션을 위한 메모리가 할당 및 초기화 되며, main 함수에 진입하기 전에 C runtime system에 의해 0으로 초기화되는 메모리로 매핑된다.

## .symtab

- 모든 정적 함수와 변수를 포함한 모든 심볼 테이블의 정보가 위치하는 섹션

## .debug

- 디버깅 심볼 포함

## .strtab

- .debug에서 사용하는 코드 데이터 포함
