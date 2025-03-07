---
title: "[임베디드] Clang 컴파일 시 BIN 파일 크기 문제 & ELF와 BIN 차이"
last_modified_at: 2025-03-07T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---

요즘 LLVM Compiler를 사용해서 졸업 연구를 진행하고 있다. 임베디드 분야가 다 그런 것 같지만, 어디서 알려주는 데도 없고, 자료도 잘 없고, 있어도 거의 해외 영문 사이트인게 참 힘들다. 그래도 그런만큼 내가 직접 원인을 찾고 수정하는 묘미가 있는 분야이다.

거두절미하고, LLVM-Compiler에서는 Clang을 Front end compiler로 두어 C/C++ 컴파일을 수행하고 있다. Clang은, LLVM이 기존 GCC 컴파일러를 사용하지 않고 LLVM 전용 Front end를 두는 것이 더 빠르다고 판단하여 만들어진 프론트엔드이다. 

문제는, 컴파일 후 실행 파일을 보드에 올리려고 했을 때 발생했다.

## .bin file is larger than .elf ??

![Image](https://github.com/user-attachments/assets/d309c030-7107-433c-bc4b-7870f11048c1)

위 사진을 보면, .bin 파일과 .elf 파일의 크기가 상당히 크다. 내가 사용하는 보드는 STM32F407로, 플래시 메모리가 1M 정도인데 elf 파일 크기는 1.1M. 설상가상으로 bin파일은 elf 파일보다 400배가 더 크다.

## elf vs bin

 예전 포스팅 ELF Symbol 설명 참조 : https://kimbongsik.github.io/embedded/2023/08/01/embedded_elf_symbol.html

예전에 포스팅 했었는데, **ELF 파일은 실행 및 링킹 가능한 파일이다.** 실행 파일임에도 파일 내부에 디버깅 정보, 함수 심볼 등에 대한 정보가 남아 있어 구조 분석 및 파싱이 비교적 수월하게 가능하다. 

바꿔 말해서,  해당 정보들은 CPU에게는 크게 필요가 없는 정보들이다. 심볼 정보, 그러니까 함수 이름이나 링킹 섹션 등은 개발자가 알아보기에 좋은 정보들이고, 디바이스에 올라갈 펌웨어에는 elf 가 잘 쓰이지 않는다.

그럼 뭘 쓰나? 바로 bin 파일이다.  **bin 파일은 이진 형식으로 인코딩된 데이터를 담고 있는 파일**이다. 주로 하드웨어에서 사용되는 ‘펌웨어 이미지’, 또는 ‘바이너리 이미지’라고 하면, 이 .bin 포맷 파일을 생각하면 된다. 이진 파일 형식이기 때문에 텍스트 에디터로 열어보면 사람은 하나도 못 알아본다. 그리고 elf 에는 있던 심볼 정보들이 없다! 때문에 bin 파일을 파싱하는 일은 매우 어렵다.

어..? 그럼 bin 파일은 elf 파일보다 작아야 하지 않나?

## Why is .bin file bigger than .elf ?

**arm-none-eabi-objdump** 를 사용해 elf로부터 bin 파일을 획득했다. 이때 elf의 심볼 정보들은 모두 지워지고 압축(실제 압축은 아니고, 표현하자면) 되었을 것이다. 그랬는데도 bin 파일이 사이즈가 훨씬 더 크다.

정답부터 말하자면, **원래 bin 파일이 elf 파일보다 작은 게 정상이다**.

그런데 bin 파일이 간혹 elf 파일보다 커지는 경우가 있는데, 바로 “RAM” 영역 때문이다.

```nasm
Index Name        Size     VMA       LMA       File off  Algn
  0 .isr_vector 000001c0 08000000 08000000  00010000  2**4
  1 .text       00028000 08000200 08000200  00010200  2**4
  2 .rodata     00002000 08028200 08028200  00038200  2**4
  3 .data       00001000 20000000 0802a200  0003a200  2**4
  4 .bss        00003000 20001000 20001000  00000000  2**4
```

‘-h’ 옵션으로 elf 섹션 정보를 확인해보자. 대충 이런 식으로 생겼을텐데, 보면 RAM 영역에 해당하는 .data 섹션과 .bss 섹션의 가상 주소(VMA)가 20000000으로, Flash 주소보다 훅 뛰는 걸 알 수 있다. 

그런데 **.bin 파일은, 메모리 주소가 연속되지 않으면 그 사이를 패딩(0)으로 채운다.**

즉, 이 상태로 objdump로 elf로부터 bin 파일을 얻어내면 bin이 0x08000000 ~ 0x20000000 사이를 0으로 채우면서 엄청난 크기가 된다는 뜻이다.

## How to solve it ?

RAM 영역이 문제라면, RAM 섹션을 제외하고 변환 시켜주면 된다.

이렇게.

```bash
arm-none-eabi-objcopy -O binary --remove-section=.bss --remove-section=.data example.elf example.bin
```

그럼 해당 섹션을 날려도 문제는 없는가? 여기서 섹션을 제거했다고 해도, 데이터가 사라지는 것은 아니다. 

### —remove-section=.bss

.bss에는 초기화되지 않은 전역 변수가 저장된다.

 .bss는 RAM에서 0으로 초기화되는 섹션이기 때문에 플래시에 저장될 필요가 없으며(보통 elf에서 사이즈만 잡혀있다.), STM32 스티트업 코드에서 자동으로 RAM에서 0으로 초기화 시킨다. 

### —remove-section=.data

.data는 초기값은 플래시에 저장되어 있다가, 부팅 시 RAM으로 복사된다. 

즉, .data의 값은 이미 플래시에 저장이 되어 있고, 마찬가지로 스타트업 코드에서 플래시에 저장된 초기값을 RAM으로 복사하는 루틴이 있어 동작할 때 문제 없다.

### 결과

![Image](https://github.com/user-attachments/assets/c3615bc0-2954-46d2-b415-a259d5690fb3)

어..? 그런데도 사이즈가 줄어들지 않는다.

## Clang과 CCM-RAM

여기서 한참 씨름 했는데, 그건 clang 때문이다.

프로젝트의 링커 스크립트를 확인해보면,

![Image](https://github.com/user-attachments/assets/3a648bd6-1ef0-4dac-bb59-2823ed172344)

여기 CCM-RAM이라는 섹션이 존재한다. 

CCM-RAM은 Core Coupled Memory로, 일부 STM32 마이크로컨트롤러에서 제공하는 고속 RAM이다. 코어와 직접 연결되어 SRAM보다 빠르다는 특정이 있다. 하지만 DMA는 사용할 수 없는 특별한 RAM이라고 보면 된다. 즉, 개발자가 해당 CCM-RAM을 사용하려고 하지 않는 이상, 저 CCM-RAM은 일반적으로 사용되지 않는다.

GCC에서는 사용하지 않는 섹션은 자동으로 제거해주는데, Clang은 그렇지 않은 것 같다..
해당 섹션을 주석 처리해주면 정상적으로 사이즈가 줄어든다.

![Image](https://github.com/user-attachments/assets/a9d61f37-7e0a-436c-a940-599c5f789fa5)

이제, bin 파일은 elf 파일보다 작다!