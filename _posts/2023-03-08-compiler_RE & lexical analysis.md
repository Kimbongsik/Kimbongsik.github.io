---
title: "[컴파일러] Lexical Analysis & Regulatr Expression"
last_modified_at: 2023-03-08T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - compiler

tags:
  - compiler
---

<br/>

# Front-end Compilation

현대의 컴파일러들은 대부분 front-end와 back-end가 확실히 나누어진다. Front-end단 처리가 끝나면 IR(Intermediate Representation) 형태로 프로그램이 생성된다. IR이란 중간언어로 생각하면 되는데, C/C++, Swift, Ruby 등의 고급 언어를 파싱하면 생성된다. LLVM이라는 프레임워크는 이 LLVM IR을 효과적으로 이용하여 최적화를 하고, back-end 과정에서 최종적으로 타겟머신에 맞는 기계어를 생성한다.  

컴파일러의 Front-end 과정은 아래와 같다.

1. **Lexical Analysis**
2. **Syntax Analysis**
3. **Semantic Analysis**
4. **IR Code Generation** 

오늘은 Front-end Compilation 과정 중 첫 번째인 Lexical Analysis에 대해 알아볼 것이다.

<br/>

# Lexical Analysis이란

컴파일러는 프로그래머가 작성한 코드, 즉 문자열들을 스캔하고 의미 있는 단위로 묶어 이른바 ‘토큰(Token)’이라는 것을 생성한다. 

<br/>

## Lexical Token

- Token이란?
    - 한덩이로 묶이는 캐릭터(char)의 집합

<br/>

Token은 token name, token value 쌍으로 구성된다.

Token들의 종류는 아래와 같다.

| Type | Examples  |
| --- | --- |
| ID | foo, i, j, n_12 |
| NUM | 12, 345, 7890 |
| REAL | 3.14 |
| IF | if |
| NOTEQ | != |
| COMMA | . |

<br/>

## Lexer

![1](https://user-images.githubusercontent.com/63995044/223706511-e15a8a4c-ef28-4665-8ec7-a0755a902b1e.png)

Lexer는 토큰들의 의미를 분석하는 역할을 수행한다. Lexical Analyze는 string을 토큰화 시키고, 해당 토큰을 Lexer를 거쳐 결과를 분석하는 과정을 통틀어 일컫는 말이다. RE(Regular Expressions)를 통해 프로그램의 토큰을 정의하고, Lexer Generator를 통해 Lexer를 생성한다.

실제 컴파일 단계는 Source → Lexer → Stream of Tokens 이 부분에서 일어난다.

<br/>

# Regular Expression(RE)

언어(language)란 string의 집합을 일컫는 말이고, string은 알파벳으로 이루어진 무한한 심볼의 시퀀스이다. 정규표현식(RE)은 언어를 표현하는, string의 집합이다. 크게 다섯 가지 basic notation을 소개하면,

1. **Symbol**
    - 문자열 a만 포함하는 언어를 나타내는 정규 표현식
    - ex: a → {a}
    - ex: 1 → {1}
2. **Epsilon:** ε
    - empty string을 의미
3. **Alternation: M|N**
    - 일종의 합집합
    - ex: a | b → {a, b}
4. **Concatenation: MN**
    - 합쳐서 만들 수 있는 모든 경우의 집합
    - ex: (a | b)(a | c) → {aa, ac, ba, bc}
5. **Repition: M***
    - 어떤 string이든 꺼내서 만들 수 있는 모든 조합(무한한 개수)
    - empty string(Epsilon)도 가능
        - * : 0 이상
    - ex: (a | b)* → {ε, a, b, aa, ab, ba, bb, aaa, aaab,,,,, }

<br/>

또, 아래와 같은 표현법이 있다. 부연 설명 없이 예시만 제시하겠다.

<br/>

- [abc] : (a | b | c)
- [b-g] : [bcdefg]
- [b-gM-Qkr]: [bcdefgMNOPQkr]
- M? : (M |  ε)
    - ? : 0 또는 1
    - M이 올 수도 있고, empty string이 올 수도 있다
    - ab? 라고 작성되었다면, a가 올 수도, ab가 올 수도 있는 것
- M+ : MM*
    - + : 1 이상
    - empty string 제거를 위함
    - 예를 들어, a+ 는 {a}, {aa,,}가 가능하며, repition과 달리 empty string이 불가능함
- .
    - any single character except newline
- “a.+”
    - 문자열 그대로

- 참고
    - 소괄호 ‘()’
        - 소괄호 안에 있는 글자를 하나의 덩어리로 인식한다
        - a , b , c , d 로 인식하게 하기 위해서는 (a | b | c | d) 와 같이 작성한다
    - 대괄호 ‘[]’
        - 대괄호 안에 있는 문자를 각각 인식한다
        - [abcd]는 a, b, c, d로 인식된다. 즉, [abcd] == (a | b | c | d)


<br/>

위 notations를 적용하면 아래와 같이 표현할 수 있다.

<br/>

- Binary numbers
    - → (0 | 1)*
- aa가 항상 포함되는 a와 b로 이루어진 스트링
    - →(a|b)*aa(a|b)*

<br/>

Lexical Token을 RE로 표현해보면 아래와 같다.

중요한 점은 RE로 모든 언어를 표현할 수 있는 것은 아니다. 예를 들어 “a와 b가 포함된 string에서 항상 a가 b 보다 많다”는 표현은 RE로 표현할 수 없다. 

| Type | Examples  |
| --- | --- |
| ID | [a-z][a-z0-9]* |
| NUM | [0-9]+ |
| REAL | ([0-9]+”.”[0-9]*) | ([0-9]*”.”[0-9]+) |
| IF | if |

<br/>

# Finite Automata

이제 정규표현식을 통해 정의된 토큰은 deterministic finite automata로 변환하는 과정이 필요하다. 그러나 바로 Deterministic Finite Automata로 넣기엔 변환 과정이 쉽지 않기 때문에 Un-Deterministic Finite Automata로 변환 후 넘긴다. 

<br/>

- 유한 상태 머신 구성 요소
    - start state
    - one or more final states
    - 상태와 상태 사이 연결을 의미하는 Edge는 한 가지 symbol로 labled 되어있다

<br/>

# Deterministic Finite Automata(DFA)

DFA는 string을 “accepts” 또는 “rejects” 된다.

- Accept
    - string이 상태머신의 마지막 상태에 도달한 상태
- Reject
    - string이 상태머신의 마지막 상태에 도달하지 못한 상태
    

![2](https://user-images.githubusercontent.com/63995044/223706524-359ae885-19e1-415b-b7ef-0e4429dda165.png)

<br/>

위와 같은 상태 머신이 있을 때, string abc를 입력으로 넣으면 어떻게 될까?

a가 입력되면, 상태 2로 넘어가게 되고, 입력으로 b가 들어왔을 때 final state에 도달하게 된다. string c가 입력으로 들어올 수 없으므로, reject된다. 그럼 아예 x와 같은 새로운 문자열이 들어온 경우는 어떻게 될까? 마찬가지로 rejecet 된다. 

<br/>

# Non-Deterministic Finite Automata(NFA)

DFA로 변환하기 전, NFA를 거친다고 했다. DFA는 가는 방향이 항상 정해져 있기 때문에, 곧바로 DFA로 변환하는 것이 어렵기 때문이다. 

NFA의 특징은 다음과 같다.

- Edges leaving a node can be identically labeled
- An edge can be labeled with ε

![3 png](https://user-images.githubusercontent.com/63995044/223706535-0fa19975-aa22-4245-9a9e-eff64b3c293d.jpg)

<br/>

NFA → DFA 변환 과정과 Lexical Analysis에 대한 더 자세한 과정은 다음 포스팅에 이어서 하도록 하겠다.