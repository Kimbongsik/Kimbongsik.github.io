---
title: "[임베디드] Makefile 작성법"
last_modified_at: 2024-05-27T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---

Makefile이 꼭 임베디드와 밀접한 관련이 있다..! 라고 할 수는 없지만..

어디에 넣기도 참 애매해서 우선 임베디드 카테고리에서 Makefile 만드는 기본 방법을 설명하려고 한다.

- Make는 파일 관리 유틸리티다
- Makefile을 사용하면 큰 프로젝트에서 여러 가지 파일을 한 번에 빌드하기 좋다
    1. gcc -o main.out main.o , gcc -o foo.out foo.o … 이렇게 여러 줄 커맨드를 쳐서 파일들을 일일이 빌드하지 않아도 된다
    2. 파일 종속성을 고려한 빌드가 가능하다. → 빌드 과정에서 변경된 소스코드에 의존성이 있는 대상들만 추려서 다시 빌드하는 기능. 이를 **Incremental build**라고 한다.
    
    예를 들어 여러 파일 중 딱 한 개의 파일만 수정하여 빌드 한다고 했을 때, 프로젝트 전체를 빌드하지 않고 바뀐 부분만 새로 빌드가 가능하다. 빌드 중 오류가 발생하여 수정했을 때도 마찬가지로, 오류가 난 이후부터 새로 빌드가 가능하다. 때문에 작업 효율성이 증가한다는 장점이 있다.
    
- **기본 규칙**

```makefile
app.out: main.o foo.o bar.o
	gcc -o app.out main.o foo.o bar.o

main.o: foo.h bar.h main.c
	gcc -c -o main.o main.c
	
foo.o: foo.h foo.c
	gcc -c -o foo.o foo.h
	
bar.o: bar.h bar.c
	gcc -c -o bar.o bar.c
```

어떤 파일을 생성하고, 해당 파일에 종속성을 가지는 파일들을 명시한 뒤 각각 맞는 컴파일 명령어들을 작성해주면 된다. 이때, 

어떤 파일을 생성하는지를 ⇒ ‘Target’,

어떤 파일들이 종속성을 가지는지를 ⇒ Dependencies

빌드 대상을 생성하는 여러 명령어들을 ⇒ Recipe 이라고 한다.

Recipe 작성 시, 반드시 앞에 Tab 문자가 포함되어야 한다.

그리고 나서,

```bash
make
```

make 명령을 사용하면, Makefile에 명시한 모든 파일들이 한 번에 생성된다.

만약 특정 파일만 빌드하고 싶은 경우, 

```bash
make bar.o
```

이와 같이 작성하면 ‘gcc -c -o bar.o bar.c’ 커맨드만 실행된다.

- **내장 규칙**

Makef에는 내장되어 있는 규칙이 몇 개 있어서, 자주 사용되는 명령어의 경우 생략이 가능하다. 

‘gcc -c -o’ 규칙이 이에 해당한다. 따라서 다음만 작성해도 잘 돌아간다.

```makefile
app.out: main.o foo.o bar.o
	gcc -o app.out main.o foo.o bar.o
```

그러나 이 경우, Make에서 헤더 파일을 감지하지 못하며, 이에 따라 헤더파일의 내용이 변경되더라도 반영되지 않는다. 따라서 이를 방지하기 위해서는 다음과 같이 써줘야 한다.

```makefile
app.out: main.o foo.o bar.o
	gcc -o app.out main.o foo.o bar.o
	
main.o: foo.h bar.h main.c
foo.o: foo.h foo.c
bar.o: bar.h bar.c
```

- **자동 변수**

더 깔끔하게 작성할 수 있는 방법들이 있다. 일반적으로 이 방법을 많이 사용한다.

다음과 같이 사용 가능.

- 자동 변수
    - $@ : 현재 target 이름
    - $^ : 현재 target이 의존하는 대상들의 전체
    - $? : 현재 target이 의존하는 대상들 중 변경된 것

```makefile
CC=gcc
CFLAG=-g -Wall
OBJS=main.o foo.o bar.o
TARGET=app.out

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) -o $@ $(OBJS)
	
main.o: foo.h bar.h main.c
foo.o: foo.h foo.c
bar.o: bar.h bar.c
```

- **Clean build**

프로젝트를 작성하다가 빌드 결과물이나 실행 중에 생성되는 중간 파일들을 모두 삭제하고 처음부터 깨끗한 상태로 다시 빌드하고 싶을 때가 있다. 

다음과 같이 Clean 매크로를 추가해주고, ‘make clean’ 명령을 사용하면 clean build가 가능하다.

```makefile
clean:
	rm -rf *.o
	rm -f $(TARGET)
```

- 참고 자료

https://velog.io/@tjeong/new-Makefile-1

https://www.tuwlab.com/ece/27193