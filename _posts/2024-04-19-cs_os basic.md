# 운영체제 서비스

> Process management
> Main memory management
> File management
> Secondary storage management
> I/O management
>    > System Call
---------------------------------

모든 app 프로그램들은 자원을 사용하므로, 이러한 자원의 효율적 분배를 위해 os가 존재한다.  
os에는 다양한 서비스들이 정부 부처처럼 존재한다.  

## Process management  
CPU 자원을 나눠주고 관리한다.
여기서 프로세스(Process)란 메모리에서 실행 중인 프로그램(program in execution)으로, CPU에 의해 실행되는 건 하드 디스크가 아니라 메인 메모리에 올라와 있는 프로그램이기 때문에
별개로 Process라 구분지어준다. 주요 기능은 다음과 같다. 

주요기능:

- 프로세스의 생성, 소멸(creation, deletion)
- 프로세스 활동 일시 중지, 재개(suspend, resume)
- 프로세스 간 통신(interprocess communication: IPC)
- 프로세스 간 동기화(symchronization)
- 교착상태 처리(deadlock handling)

## Main memory management  
주기억 장치 관리.

주요기능:
- 프로세스에게 메모리 공간 할당(allocation)
- 메모리의 어느 부분이 어느 프로세스에게 할당 되었는지 추적 및 감시
- 프로세스 종료시 메모리 회수(deallocation)
- 메모리를 효과적으로 사용
- 가상 메모리 기술(실제 물리적인 메모리보다 큰 용량을 갖추도록 하는 기술)

## File management
하드 디스크 안의 파일을 관리. Track과 sector로 구성된 디스크를 파일이라는 논리적인 관점으로 보게 한다. 

주요 기능:
- 파일 생성과 삭제
- 디렉토리의 생성과 삭제
- 기본 동작 지원: open, close, read, write, create, delete
- Track/Sector ->파일간의 매핑
- 백업

## Secondary storage management
보조 기억 장치 관리. 하드 디스크, 플래시 메모리 등이 해당된다. 
주요 기능:
- 빈 공간 관리(free space management) *sector들을 몇 개씩 모아 block이라 하는데, 여기에 데이터를 저장한다. 여기에서 비어져 있는 공간을 관리한다. 
- 저장공간 할당(storage allocation)
- 디스크 스케줄링 *head를 효율적으로 움직여 track/sector를 읽음

## I/O management
입출력 장치 관리.
주요 기능:
- 장치 드라이브(device drivers) *하드웨어와 os가 만나는 부분에 device driver들이 있다. 
- 입출력 장치의 성능향상:
* buffering : 입출력 장치에서 읽은 내용을 메모리로 들고 옴. 빠르게 읽을 수 있음.
* spooling : 메모리 대신 하드 디스크를 중간 매체로 사용한다. 프린트는 일반적으로 속도가 느린데, 처리될 때까지 cpu가 계속 기다릴 수는 없으므로, 프린트 보다 빠르고 CPU보다는 느린 디스크에 저장을 해두고 이 내용을 천천히 프린트로 내보낸다.)

그 외에도 OS는 networking이나 protection 등의 서비스를 제공한다. 

## 시스템 콜(System call)

Application 프로그램이 os가 제공하는 서비스를 받기 위해 호출하는 것. 
주요 시스템 콜:
-Process: end(종료), abort(강제 종료), load(하드 디스크에 있는 프로그램을 메모리로 가져옴), execute, create(프로세스 생성), terminate, get/set attributes, wait event, signal event
-Memory: allocate(어떤 프로그램이 돌면서 객체를 만들기 위해 메모리를 요청), free(메모리를 다 사용한 후 돌려놓음)
-File
-Device
-Information

* MS-DOS는 어떻게 시스템 콜을 했을까? 다음은 DOS function임.

--------------------------------------------------------

**AH = 3Ch - "CREAT" - CREATE OR TRUNCATE FILE**

Entry:

- CX = [file attributes](http://spike.scu.edu.au/~barry/interrupts.html#fattr)
- DS:DX -> ASCIZ filename (*DS register와 DX register가 파일의 이름을 나타내도록 해라)

Return:

- CF clear if successful, AX = file handle
- CF set on error AX = error code (03h,04h,05h)

Notes: if a file with the given name exists, it is truncated to zero length

SeeAlso: AH=16h,AH=3Dh,AH=5Ah,AH=5Bh

------------------------------

메모리 주소 100번지에 "AAAA"라는 이름의 파일을 생성한다고 해보자. 
그에 따른 assembly 코드는 다음 예시와 같다. (file attribute는 0으로 두자)  

mov cx, 0  
mov dx, 100  
mov eh, 3c  
int 21h  


* Linux  
mov eax, 0  
mov ecx, 0  
mov edx, (이름이 적혀있는 주소)  
int 80h  


-> 하드 디스크에 파일을 생성하기 위해 정해진 약속대로 이와 같이 os에게 명령을 내려 생성할 수 있다. 
