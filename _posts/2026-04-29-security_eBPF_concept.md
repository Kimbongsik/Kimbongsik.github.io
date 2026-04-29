---
title: "[보안] 한 눈에 보는 eBPF"
last_modified_at: 2026-04-29T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - security

tags:
  - security
---

# 1. 정의 및 구성

- 운영체제 커널과 같은 특별한 권한이 있는 환경에서 샌드박스 프로그램을 실행시킬 수 있는 리눅스 커널 기술
- 커널 소스 코드를 바꾸거나 커널 모듈을 로드하지 않고도 런타임에 기존 커널 기능을 확장 가능
- eBPF 프로그램은 이벤트 기반으로 동작하며, 커널 또는 어플리케이션이 특정 훅 지점을 지나갈 때 실행됨
- 사전 정의 된 훅은 시스템 콜, 함수 진입 및 종료, 커널 tracepoint, 네트워크 이벤트 등이 포함될 수 있음

![이벤트 훅을 사전 정의 해두었을 경우](/assets/images/posts/1.png)

이벤트 훅을 사전 정의 해두었을 경우

- 또는, kprobe 및 uprobe를 생성하여 커널 또는 유저 어플리케이션에 eBPF 프로그램을 부착할 수 있음

![커널 요소에 eBPF 프로그램을 부착한 경우](/assets/images/posts/2.png)

커널 요소에 eBPF 프로그램을 부착한 경우

# 2. eBPF 프로그램 작성

- Cilium, bcc, bpftrace와 같이 eBPF 추상화 프로젝트를 사용하는 경우가 많음
    - 사용자 의도에 따라 eBPF 프로그램을 자동으로 구현하는 기능을 제공함
- 고수준 추상화를 사용할 수 없는 경우, eBPF 프로그램을 직접 작성해야 하며, LLVM Compiler를 통해 Bytecode 형태로 작성함

![eBPF 동작 구조](/assets/images/posts/3.png)

# 3. 동작 시퀀스

1. C로 eBPF 프로그램 작성
2. clang으로 bytecode 생성
    - 일반적인 C 코딩이 아닌, bpf helper를 사용하는 제한된 C 코딩
        - 예시)
        
        ```c
        // trace_open.c
        #include <linux/bpf.h>
        #include <bpf/bpf_helpers.h>
        
        SEC("tracepoint/syscalls/sys_enter_openat") // ATTACH 위치 정의
        int trace_open(struct trace_event_raw_sys_enter *ctx)
        {
            bpf_printk("openat called.\n"); // 커널 trace pipe로 출력
            return 0;
        }
        
        char LICENSE[] SEC("license") = "GPL"; // GPL 라이선스
        ```
        
    - clang 컴파일러로 컴파일하며, 해당 bytecode는 eBPF **Program + Map** 정의가 포함된 object 파일임
    - **즉, CPU용 바이너리가 아닌 eBPF VM용 bytecode**
    - 이때 Program은 실제 실행될 로직이며, Map은 사용자 공간과 커널 공간 사이 커널 상태를 안전하게 상태를 저장할 수 있는 데이터 공유용 구조임
3. 사용자 공간 프로그램에서 bpf() syscall 호출
    - 작성한 bytecode를 커널에 넣어주는 로더 프로그램들이 존재함
        - bcc, libbpf, Go eBPF library ..
        - libbpf 방식으로 로더를 작성할 경우 예시)
        
        ```c
        // loader.c
        #include <bpf/libbpf.h>
        
        int main()
        {
            struct bpf_object *obj;
            obj = bpf_object__open_file("trace_open.o", NULL);
            bpf_object__load(obj); //내부적으로 bpf() syscall 호출
            return 0;
        }
        ```
        
4. eBPF Verifier 검사
    - 커널에 bytecode가 로드될 때 무한 루프부, 포인터 안전성, 커널 메모리 침범 등 검사 수행
    - 두 단계를 통해 검사
        1. CFG 검사
            - 무한 루프 존재 여부
            - 도달 불가능한 코드
        2. Register State Tracking
            - 레지스터의 타입과 값 범위를 추적
            - 모든 명령어를 실제 가상의 Verifer 내부에서 시뮬레이션하여 레지스터와 스택에 어떤 변화가 있는지 검사
    - 예시)
        
        ```c
        // 허용 안 되는 것
        while(1) {} // 무한루프
        int arr[1024]; // 큰 스택 할당
        char *ptr = malloc(100); // 동적 메모리 할당(힙 X)
        print("hello\n"); // 표준 라이브러리 함수 호출
        int func_a() { return func_b(); } // 일반 함수 포인터 간접 호출
        int *p = NULL; return *p; // NULL dereference
        
        ```
        
5. JIT 컴파일 (옵션)
    - 켜져 있는 경우, eBPF bytecode → ARM code로 변환됨
        - 오버헤드가 낮아 임베디드 환경에서 사용하기 적합
        - 다만, ARM32 환경(Zynq-7035 ARM Cortex-A9도 해당)에서는 제약이 있을 수도 있으므로 개발 시 고려해야 함
            - eBPF는기본적으로 64bit 레지스터 ISA임
            - 즉 ARM32환경에서는 연산 분해(64bit 레지스터를 32bit 레지스터 2개로 표현)가 필수로 동반됨
            - 또한, eBPF에서 사용되는 레지스터가 총 10개이므로, 개수가 부족할 수 있어 스택 spill, reload가 발생할 수 있음
                - 1 cycle 수준에 접근할 수 있는 데이터를(레지스터로 연산할 경우), 수십 cycle의 메모리 접근으로 변환해야 하는 것
            - 따라서, 포인터 연산, map access, 조건문이 많은 경우 JIT를 사용해도 문제 없을지 검토가 필요할 것
    - 꺼져 있는 경우, 인터프리터 모드로 동작함
        - 이벤트 하나 당 수백-수천 사이클의 오버헤드가 발생할 수 있으므로, 성능을 고려해야 함
6. 특정 Hook 시점에 attach
    - 위 코드 예시와 같이 SEC(”tracepoint/syscalls/sys_enter_openat”)과 같이 정의해두면, 시스템콜 진입 시점에 자동 실행됨
    - Tracepoint attach 내부 흐름
    
    ```c
    1. /sys/kernel/debug/tracing/events/syscalls/sys_enter_execve/id 읽기
    -> tracepoint의 커널 내부 ID 획득 (ex: id == 42)
    2. perf_event_open(PERF_TYPE_TRACEPOINT, id=42) syscall
    -> perf_event fd 반환
    3. ioctl(perf_fd, PERF_EVENT_IOC_SET_BPF, prog_fd)
    -> perf 이벤트에 BPF 프로그램 연결
    4. ioctl(perf_fd, PERF_EVENT_IOC_ENABLE, 0)
    -> 이벤트 활성화 후, 트리거 발생 시 BPF 실행
    ```
    
    - Kprobe attach 내부 흐름
        - 
7. 이벤트 발생 시 자동 실행
    - 유저 프로세스에서 open()을 호출할 시, 아래와 같은 시퀀스를 따름
        1. Process → syscall openat()
        2. tracepoint 발생
        3. eBPF 프로그램 실행
        4. bpf_printk 실행
    - 이후 /sys/kernel/debug/tracing/trace_pipe 에서 “openat called!” 메시지를 확인할 수 있음
8. Map으로 사용자 공간 ↔ 커널 데이터 교환
    - 위와 같이 printk를 사용하지 않고, Map을 사용할 수도 있음
        - 예시)
            
            ```c
            struct {
                __uint(type, BPF_MAP_TYPE_HASH);
                __uint(max_entries, 1024);
                __type(key, u32);
                __type(value, u64);
            } counter SEC(".maps");
            
            SEC("tracepoint/syscalls/sys_enter_openat")
            int trace_open(struct trace_event_raw_sys_enter *ctx)
            {
                u32 key = 0;
                u64 init_val = 1;
                u64 *val;
            
                val = bpf_map_lookup_elem(&counter, &key);
                if (val) {
                    __sync_fetch_and_add(val, 1);
                } else {
                    bpf_map_update_elem(&counter, &key, &init_val, BPF_ANY);
                }
            
                return 0;
            }
            ```
            
- 해당 map은 커널 공간에 존재하며, 유저 프로그램이 bpf_map_lookup_elem() 함수를 통해 읽을 수 있음
- 위 구조체는 컴파일되면 ELF 파일의 .maps 섹션에 포함되며, 로더(libbpf)가 이를 읽어 커널에 실제 map 객체를 생성함
- eBPF 로더에서 내부적으로 bpf(BPF_MAP_CREATE, ..) syscall 호출함
    - map, key, value, 등을 확인한 뒤 메모리 할당 + map 객체 생성 수행
- 커널 공간에 존재하는 map은 사용자 프로그램에서 file descriptor를 통해 접근할 수 있음
    - 예시)
        
        ```c
        int map_fd = bpf_object__find_map_fd_by_name(obj, "counter");
        
        u32 key = 0;
        u64 value;
        
        bpf_map_lookup_elem(map_fd, &key, &value); // bpf(BPF_MAP_LOOKUP_ELEM, ..)
        
        printf("openat count: %llu\n", value);
        ```
        

# 4. eBPF Verifier

- eBPF는 기본적으로 10개의 레지스터(R0~R9)를 사용하는 VM임
    - 즉, 이때 말하는 레지스터는 실제 레지스터가 아닌, 실행 전에 시뮬레이션 하기 위한 추상 상태 변수라고 볼 수 있음
- Verifer는 각 레지스터를 아래와 같이 봄
    
    ```bash
    (reg type, value range, pointer metadata)
    ```
    
- 그리고 Verifier는 레지스터의 타입과 값 범위를 추적하여 검사를 수행함
- 아래는 대표적인 eBPF 레지스터 타입
    
    ```bash
    - NOT_INIT: 초기화되지 않음
    - SCALAR_VALUE: 일반 정수값
    - PTR_TO_CTX: 훅 컨텍스트 포인터
    - PTR_TO_MAP_VALUE: Map lookup 결과 포인터
    - PTR_TO_STACK: 스택 포인터
    - PTR_TO_PACKET → 패킷 데이터
    ```
    
- eBPF Verifier의 목표는, 모든 실행 경로에서 안전함을 증명하기 위함임
    - 어떤 입력, 분기, 값이 들어와도 잘못된 메모리 접근이나 무한 루프, 커널 침범이 없도록 하기 위해 설계됨

# 5. perf

- 리눅스 커널의 이벤트 수집 및 전달을 담당하는 인프라
- 다양한 이벤트들을 하나의 통합된 인터페이스로 처리할 수 있음
    - tracepoint, kprobe, uprobe 등
- 역할은 다음과 같음
    - 커널에서 발생한 이벤트를 수집
    - 이벤트를 user space 또는 eBPF 프로그램으로 전달
    - 이벤트 기반으로 특정 동작(eBPF 실행 등)을 트리거 함
        
        ```bash
        tracepoint / kprobe / uprobe
                ↓
              perf
                ↓
           eBPF / user space
        ```
        
- perf는 이벤트를 관리하기 위해 perf_event라는 구조체를 사용하는데, 각 attach마다 하나의 perf_event가 생성됨
- perf_event는 아래의 정보들을 포함함
    - 어떤 이벤트인지(tracepoint, kprobe 등)
    - 연결된 eBPF 프로그램
    - ring buffer 정보
- tracepoint를 사용했을 때 대략적인 attach 시퀀스를 보면,
    
    ```bash
    1. tracepoint ID 확인
    : /sys/kernel/debug/tracing/events/.../id
    
    2. perf_event_open() 호출
    : perf_event 생성
    
    3. ioctl(PERF_EVENT_IOC_SET_BPF, prog_fd)
    : eBPF 연결
    
    4. 이벤트 발생 시 eBPF 실행
    ```
    

# 6. Attach Point

## Tracepoint란?

- 커널이 미리 만들어둔 hook으로, 미리 정의된 지점임
- 정확히는, 커널 코드에 미리 심어둔 “이벤트 발생 지점”이자, 그 이벤트를 외부에 공개하는 “시스템”에 가까움
- 커널 코드 안에는 아래와 같은 데이터가 포함되어 있음
    
    ```c
    trace_sys_enter_openat(dfd, filename, flags, mode);
    ```
    
    - 이 데이터가 tracepoint이며, 이곳에서 이벤트를 발생시킨다는 명시적인 함수 호출임
    - **eBPF는 이 이벤트가 발생했을 때 실행할 코드를 등록함**
    - openat() 호출 시 실행 메커니즘 예시(아래서 재설명)
    
    ```c
    1. user process에서 openat() 호출
    2. kernel 진입
    3. tracepoint 호출
    -> trace_sys_enter_openat(...)
    4. tracepoint subsystem
    -> 등록된 listener 확인
    5. perf_event layer
    -> 연결된 eBPF 프로그램 확인
    6. eBPF 실행
    ```
    
- 특히 tracpoint는 커널에 이미 attach 지점과 넘기는 데이터들이 정의 되어 있어 구조가 안정적이라는 특징이 있음
    - kprobe와 비교 했을 때, kprobe는 데이터들에 대한 정의가 되어 있지 않아 어떤 값이 인자인지 직접 해석해야 하는 불편함이 있음
    - 즉, syscall에 대한 인자가 정리되어 있어 안정적으로 유지 되며, 커널 버전이 바뀌어도 비교적 안전함
- 예를 들어, sys_enter_openat이라는 tracepoint를 보면, 아래와 같은 구조체를 저장하고 있음
    
    ```c
    struct {
    	int dfd;
    	const char *filename;
    	int flags;
    	int mode;
    }
    ```
    
- 해당 데이터가 perf로 전달될 때는 아래와 같은 공통 포맷(구조체)로 저장됨
    
    ```c
    struct trace_event_raw_sys_enter{
    	long id;
    	unsigned long args[6];
    }
    ```
    
    - perf-eBPF 인터페이스에서는 모든 syscall을 하나의 포맷으로 처리해야 하므로, arguments 형식으로 데이터들을 관리함
    
    ```c
    args[0] = dfd
    args[1] =filename
    args[2] = flags
    args[3] = mode
    ```
    
- 다시 실행 흐름을 정리하면 이렇게 정리할 수 있음
    
    ```c
    1. user process에서 openat() 호출
    : openat(dfd, filename, flags, mode);
    : syscall(SYS_openat, ...);
    
    2. kernel 진입
    : __do_sys_openat2(...)
    
    3. tracepoint 호출
    // kernel/trace/trace_syscalls.c
    : trace_sys_enter(regs,id);
    : trace_sys_enter_openat(dfd, filename, flags, mode);
    : 내부적으로는 매크로로 정의되어 있고, 이는 커널 설정에서 tracepoint가 활성화된 경우에만 실행됨
    : 매크로 안에서 아래와 같은 구조체를 만들고 값을 채움
    
    struct trace_event_raw_sys_enter{
    	struct trace_entry ent;
    	long id; // openat
    	unsigned long args[6];
    }
    
    : 그리고, 아래와 같이 args[] 배열로 변환  
    entry->args[0] = dfd;
    entry->args[1] = filename;
    entry->args[2] = flags;
    entry->args[3] = mode;
    
    4. tracepoint subsystem
    : 등록된 listener(여기서는 perf) 확인
    
    5. perf_event layer
    : eBPF를 attach하면 perf가 listener로 등록됨
    : tracepoint 데이터(구조체 등. ctx라는 객체로 정의)를 perf_event로 전달
    : 즉, tracepoint 매크로 내부에서 구조체를 생성하고,
    : perf는 그 포인터를 그대로 전달 전달
    : bpf_prog_run(ctx)
    
    6. eBPF 실행
    : perf로부터 전달 받은 ctx를 통해 커널 데이터에 접근
    : ctx->args[] 접근
    : 그래서 eBPF 프로그램을 작성할 때,
    
    SEC("tracepoint/syscalls/sys_enter_openat") // ATTACH 위치 정의
    int trace_open(struct trace_event_raw_sys_enter *ctx)
    {
    		int dfd = ctx->args[0];
    		const char *fiename = (const char *)ctx->args[1];
        bpf_printk("openat called.\n"); // 커널 trace pipe로 출력
        return 0;
    }
    
    : 이와 같이 ctx로 syscall 인자에 접근할 수 있음
    : 여기서 ctx는 tracepoint에서 만든 entry의 주소와 동일함
    
    ```
    

## Kprobe이란?

- 커널 함수에 사용자가 원하는 지점에 붙일 수 있도록 하는 기능
- 커널 함수 시작 지점 또는 특정 명령어에 breakpoint를 삽입해서 실행을 가로챔
- 동작 흐름을 보면 다음과 같음

```c
1. do_sys_open 함수 시작 주소 찾음
2. 첫 명령어를 breapoint로 교체
3. 함수 실행 시 trap 발생
4. 커널이 kprobe handler 실행
5. eBPF 실행
6. 원래 명령어 실행
7. 정상 흐름 복귀
```

- eBPF 프로그램 작성 예시는 아래와 같음

```c
SEC("kprobe/do_sys_open")
int kprobe_open(struct pt_regs *ctx){
	bpf_printk("do_sys_open called\n");
	return 0;
}
```

- 위 tracepoint 예제와 달리, pt_regs *ctx로 포인터를 넘기는 것을 볼 수 있음
    - tracepoint는 이미 구조화된 데이터가 있어 이를 넘겨주고, kprobe는 단순히 CPU 레지스터 상태만 넘겨줌

- 유저 프로그램 함수에 붙일 수 있는 기능
- 유저 프로그램의 함수에 breakpoint 삽입
- 동작 흐름을 보면 다음과 같음

```c
1. 유저 바이너리 주소 계산
2. 해당 instruction을 breakpoint로 교체
3. 실행 시 trap -> 커널 진입
4. uprobe handler 실행
5. eBPF 실행
6. 원래 명령어 실행
7. 정상 흐름 복귀
```

- eBPF 프로그램 작성 예시는 아래와 같음. Kprobes와 동일하게 ctx 값은 CPU trap context를 전달하기 위한 CPU 레지스터값만 포함

```c
SEC("uprobe//usr/bin/drone_ctrl:0x1234") // /usr/bin/drone_ctrl의 오프셋 0x1234에 attach
int uprobe_open(struct pt_regs *ctx){
	u64 arg0 = PT_REGS_PARM1(ctx);
	bpf_printk("user open called, arg0=%llu\n", arg0);
	return 0;
}
```

- 특징 및 활용 예시
    - 유저 공간 바이너리의 특정 함수에 attach
    - 라이브러리 함수 추적에 유용함(ex. libc의 malloc, SSL 함수)
    - 바이너리 오프셋 기반으로 동작하여, 소스코드 없이도 동작함
    - 특정 어플리케이션 모니터링에 사용 가능
        - 통신 라이브러리, 위성 제어 프로세스 특정 함수 호출 감시, openssl handshake 모니터링 등

## LSM(Linux Security Module) Hook이란?

- Linux 안에서 여러 SEcurity Module들의 구동 환경을 제공해주는 Security Framework
    - SELinux, Capability, smack 등의 기법들 모두 LSM을 이용함
    - Linux Kernel Code 내부에 Hook을 넣어 커널이 Security Module의 함수를 호출할 수 있게 만드는 인터페이스 역할을 수행함
- 보안 기능에 초점이 맞추어져, 보안 결정을 내리기 직전 시점에 hook을 걺
- **감시 뿐 아니라 차단도 가능한 특징으로 인해 능동적 보안 가능**
- Kernel 5.7 이상, CONFIG_BPF_LSM을 활성화 해야 함
- 특징 및 활용 예시
    - bpf_lsm_file_open
        - 파일 open() 호출
        - 민감 파일 접근 및 감사 & 차단에 활용
    - bpf_lsm_bprm_check_security
        - execve() 프로세스 실행
        - 실행 파일 서명 검증
    - bpf_lsm_task_kill
        - 프로세스 시그널 전송
        - 시그널 기반 DoS 방지

# 7. 크로스 컴파일 및 배포(개발 참고)

- 전체 빌드 파이프라인
    
    ```bash
    개발 머신 (Ubuntu)
    │
    ├─ clang --target=bpf → execve_monitor.o   (BPF 오브젝트, CPU 무관)
    │
    └─ arm-linux-gnueabihf-gcc → loader        (ARM32 유저공간 바이너리)
       (PetaLinux SDK toolchain 사용)
    │
    타겟 보드
    │
    ├─ execve_monitor.o  → /opt/ebpf/
    └─ loader            → /opt/ebpf/
    
    실행:
      ./loader  ← ARM32 바이너리, BPF 오브젝트를 커널에 로드
    ```
    
- Makefile 예시(크로스 컴파일)
    
    ```bash
    # BPF 오브젝트 빌드 (타겟 독립적)
    CLANG    := clang
    BPFTOOL  := bpftool
    ARCH     := arm
    
    # ARM32 크로스컴파일러 (PetaLinux SDK)
    CROSS    := arm-linux-gnueabihf-
    CC       := $(CROSS)gcc
    
    BPF_CFLAGS := -g -O2 -target bpf \
                  -D__TARGET_ARCH_arm \
                  -I./vmlinux/
    
    # BPF 오브젝트 컴파일
    %.bpf.o: %.bpf.c vmlinux.h
    	$(CLANG) $(BPF_CFLAGS) -c $< -o $@
    
    # Skeleton 헤더 생성
    # bpftool이라는 도구가 BPF 오브젝트 파일을 분석해서 자동 생성하는 타입이 안전한 C 헤더임
    # bpftool gen skeleton execve_monitor.bpf.o > execve_monitor.skel.h
    # 이런 식으로 생성할 수 있는데, eBPF 프로그램을 쉽게 로드, 실행, 괸리하게 해주는 코드라고 보면 됨
    %.skel.h: %.bpf.o
    	$(BPFTOOL) gen skeleton $< > $@
    
    # 유저공간 로더 크로스컴파일
    loader: loader.c execve_monitor.skel.h
    	$(CC) -o $@ $< -lbpf -lelf -lz
    ```
    

# 8. 이외 개발 도구(CO-RE, bpftool)

## CO-RE란?

### CO-RE 정의

- eBPF 프로그램을 한 번 컴파일 하면 **여러 커널 버전에서 수정 없이 실행 가능하도록** 하는 편의 기술
- eBPF 프로그램을 사용할 때, 커널 내부 구조체에 접근하는 경우가 많음
- 그런데 커널 버전마다 구조체의 필드가 다른 경우가 있음
    - 예를 들어서..
    
    ```c
    // 커널 5.x의 task_struct (가짜 예시)
    struct task_struct {
        unsigned int flags;   // offset: 8
        pid_t pid;            // offset: 16  ← 여기
        ...
    };
    
    // 커널 5.1x의 task_struct
    struct task_struct {
        unsigned int flags;   // offset: 8
        unsigned int ptrace;  // offset: 12  ← 새 필드 추가됨!
        pid_t pid;            // offset: 20  ← 밀렸음
        ...
    };
    ```
    
- 여기에서 구조체에 접근하기 위해 하드코딩을 하게 되면,
    
    ```c
    pid_t pid = *(pid_t *)((char *)task + 16);
    ```
    
    - 위 버전에서는 맞지만, 아래 버전에서는 틀린 값을 읽게 됨
- **위와 같은 이슈를 해결하기 위해 나온 기술이 CO-RE이며, BTF와 Relocation을 통해 위 문제를 해결함**
- 대략적인 방법은 다음과 같음
    - 빌드 시점
        - clang이 BPF 오브젝트 안에 “task_struct의 pid 필드를 읽어라”는 의도를 실제 숫자 오프셋 대신 relocation 정보로 저장
    - 로드 시점
        - libbpf가 타겟 커널의 BTF를 읽어 현재 커널에서 pid 필드의 실제 오프셋을 계산
        - bytecode의 오프셋 값을 그 ㄱ밧으로 패치
        - 패치된 byteode를 커널에 로드

### BTF(BPF Type Format)

- BTF는 커널 전체의 타입 정보를 저장한 메타데이터임
- 커널이 빌드될 때 생성되며, 보통 아래 경로에 저장됨
    
    ```c
    /sys/kernel/btf/vmlinux
    ```
    
    - 이 파일 하나에 커널 전체의 모든 타입 정보가 다 들어 있음
- 이 BTF가 있어야만 CO-RE가 동작함
- 아래 옵션이 필요할 것임
    
    ```c
    CONFIG_DEBUG_INFO_BTY=y
    CONFIG_DEBUG_INFO=y
    CONFIG_DEBUG_INFO_REDUCED=n
    ```
    

### vmlinux.h

- vmlinux.h 파일은 BTF로 만드는 올인원 헤더라고 보면 됨
- CO-RE를 쓰면, 수십 개의 리눅스 커널 헤더 #include를 단 한 줄로 대체할 수 있음
- 이렇게
    
    ```bash
    # 이렇게 헤더 파일을 생성해주면
    bpftool btf dump file vmlinux_test format c > vmlinux.h
    
    # CO-RE 없이는 
    #include <linux/sched.h>
    #include <linux/fs.h>
    #..등등
    
    #그런데 CO-RE 방식으로 쓸 때는
    #include "vmlinux.h" 
    
    -> 한 줄로 정의 가능함
    ```
    

### 동작 시퀀스

- 개발 머신
    1. eBPF C 코드(BPF_CORE_READ 사용)
    2. clang -target bpf -g
        1. BPF bytecode (.text section)
            - 오프셋 자리에 임시값 + relocation 태그 삽입
        2. .BTF (.BTF.ext section)
            - task→pid 읽기 와 같은 의도 정보 저장
- 보드에서 로드 시
    - libbpf가 ELF 파일 열기
        1. .BTF.ext의 relocation 목록 파싱
            - 오프셋 n 자리에 task_struct.pid 오프셋을 넣어라
        2. /sys/kernel/btf/vmlinux 읽기
            - 현재 커널의 task_struct.pid 실제 오프셋 계산
        3. bytecode 패치
            - 임시값(0) → 실제 오프셋 값으로 교체
        4. bpf
            - 패치된 bytecode 로드

## bpftool 이란?

- bptftool은 리눅스 커널의 eBPF 상태를 조회하고 조작하는 공식 CLI 도구임
- 리눅스 커널 안에 포함된 유틸리티로, tools/bpf/bpftool 경로에 있는 커널과 함께 배포되는 공식 도구임
- 개발, 디버깅, 배포 세 상황 모두에서 쓸 수 있음
- 핵심 기능들
    1. Skeleton 생성
        - skeleton은 위에서 아주 잠시 설명했는데, bpftool이 BPF 오브젝트 파일을 분석해서 자동 생성하는 타입이 안전한 C 헤더임
        - 원래 eBPF를 쓰려면 elf 열고, map 찾고, 로드하고 attac하고 전부 직접 코드로 작성해줘야 하는데, 이 일련의 과정들을, 그냥 skeleton을 쓰면 아주 간편하게 수행할 수 있음
        - 자동으로 필요한 코드들을 내부적으로 생성해주기 때문
        
        ```c
        struct my_bpf *skel;
        
        skel = my_bpf__open();
        my_bpf__load(skel);
        my_bpf__attach(skel);
        ```
        
    2. 런타임 검사 및 디버깅
        - 아래와 같은 도구들이 있음 (claude 참조)
        
        ```c
        # ── 로드된 BPF 프로그램 목록 ──────────────────────────────
        bpftool prog list
        # 출력 예시:
        # 42: tracepoint  name trace_execve  tag a1b2c3d4e5f6
        #     loaded_at 2024-01-01T12:00:00  uid 0
        #     xlated 256B  jited 512B  memlock 4096B
        
        # ── BPF 어셈블리 덤프 (Verifier 통과 후 최적화된 코드) ───
        bpftool prog dump xlated id 42
        # 출력 예시:
        #  0: (85) call bpf_get_current_pid_tgid#14
        #  1: (77) r1 >>= 32
        #  2: (63) *(u32 *)(r10 -4) = r1
        #  ...
        
        # ── JIT 컴파일된 실제 ARM 코드 덤프 ─────────────────────
        bpftool prog dump jited id 42
        # 출력 예시 (ARM32 네이티브 코드):
        #  push {r4, r5, r6, r7, r8, lr}
        #  bl 0xc0123456 <bpf_get_current_pid_tgid>
        #  lsr r0, r0, #32
        #  ...
        
        # ── Map 목록 ────────────────────────────────────────────
        bpftool map list
        # 15: ringbuf  name rb  flags 0x0
        #     key 0B  value 0B  max_entries 4194304
        
        # ── Map 내용 덤프 (HASH map) ────────────────────────────
        bpftool map dump id 15
        # key: 00 00 00 00  value: 2a 00 00 00 00 00 00 00
        # (PID 0 → 카운트 42)
        
        # ── 현재 attach된 훅 포인트 확인 ────────────────────────
        bpftool net list
        # xdp:
        # tc:
        # flow_dissector:
        
        # ── BTF 타입 정보 조회 ──────────────────────────────────
        bpftool btf dump file /sys/kernel/btf/vmlinux | grep -A 10 "task_struct"
        # → task_struct 구조체의 모든 필드와 오프셋 확인 가능
        
        # ── 특정 프로그램이 사용하는 Map 목록 ────────────────────
        bpftool prog show id 42 --json | jq '.map_ids'
        
        ```
        

## 결론

- 즉, 정리하면 CO-RE와 bpftool은 개발 시 필수는 아님
- 다만, 아래 문제들을 해결할 때 쓸 수 있는 아주 용이한 도구
- CO-RE가 필요할 때
    - 개발 머신에서 빌드하고 보드에서 실행할 때
        - 커널 구조체 오프셋이 달라서 틀린 값을 읽을 수 있음
        
        → 만약 직접 clang으로 컴파일해서 보드에서 실행하면 커널 버전이 완전히 동일하므로 CO-RE가 없어도 됨
        
- bpftool이 필요할 때
    - 로드된 BPF 프로그램 디버깅 할 때
    - Skeleton 헤더 자동으로 생성해서 할 때
    
    → 없더도 libbpf 수동 API로 똑같이 할 수는 있음
    
    → 하지만 개발 편의성은 좋아짐