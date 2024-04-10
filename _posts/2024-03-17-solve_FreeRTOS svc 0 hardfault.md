---
title: "[임베디드] FreeRTOS 포팅 시 svc 0 HardFault Error 해결 방법"
last_modified_at: 2024-03-17T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - embedded

tags:
  - embedded
---

- FreeRTOS를 STM32 보드에 포팅하여 실행 시켰다!
- 그런데 태스크를 생성하는 코드를 돌렸을 때 SVC call에서 오류가 발생하는 경우가 있다
- 해당 오류를 해결하는 글이 한국인 중에 없어서 포스팅을 해본다
    - 사실 외국에도 별로 없다 낄낄

```c

#include "main.h"
#include "usb_host.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include "FreeRTOS.h"
#include "task.h"

..

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART3_UART_Init(void);

static void task1_handler(void* parameters);
static void task2_handler(void* parameters);

int _write(int file, char *ptr, int len)
{
  (void)file;
  int DataIdx;

  for (DataIdx = 0; DataIdx < len; DataIdx++)
  {
	  char temp = '\r';
	  if(*ptr == '\n'){
		  HAL_UART_Transmit(&huart3, &temp, 1, 65535);
	  }
	  HAL_UART_Transmit(&huart3, ptr++, 1, 65535);
  }
  return len;
}

int main(void)
{
  /* USER CODE BEGIN 1 */
  TaskHandle_t task1_handle;
  TaskHandle_t task2_handle;

  BaseType_t status;

  HAL_Init();

  SystemClock_Config();

  MX_GPIO_Init();
  MX_USART3_UART_Init();

  status = xTaskCreate(task1_handler, "Task-1", 256, "Hello Task-1", 2, &task1_handle);
  status = xTaskCreate(task2_handler, "Task-2", 256, "Hello Task-2", 2, &task2_handle);

  vTaskStartScheduler();

  while (1)
  {
    MX_USB_HOST_Process();
}

.. 
}

static void task1_handler(void* parameters)
{
  //while (1)
  //{
  	//vTaskDelay(1);
  	printf("%s\r\n", (char*)parameters);

  //}
}

static void task2_handler(void* parameters)
{
  //while (1)
  //{
  	//vTaskDelay(1);
  	printf("%s\r\n", (char*)parameters);
  //}
}

void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
}

```

- 대충 구조는,
    - priority가 동일한 태스크를 두 개 생성하고 (xTaskCreate)
        - Task1: while(1) { Hello Task1 출력; }
        - Task2: while(1) { Hello Task2 출력; }
    - 스케줄러 실행 (vTaskStartScheduler)
    

그래서 결국 RR Priority로 동작하니, Task1과 Task2가 번갈아가면서 실행되기 때문에 Hello Task1과 Hello Task2가 반복적으로 출력되어야 한다.

그런데, 프로그램이 실행되지 못하고 vTaskStartScheduler에서 Hard Fault error가 발생해 뻗어버렸다!

디버깅 해서 코드를 추적해보니,

- vTaskStratScheduler → xPortStartScheduler → prvPortStartFirstTask

여기에서, prvPortStartFirstTask는 아래와 같은 어셈블리어로 작성되었다.

![1](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/aa8c446f-6caa-48d0-bb95-6ee567538b1e)

여기에서, “svc 0”에서  hardfault error가 발생한다.

뭔가 권한 문제로 뻗은 것 같은데, 좀 더 확인해보자.

해결하기 위해서는 전체적인 맥락을 봐야 한다. 우선 prvPortStartFirstTask를 호출하게 만든 vTaskStartScheduler(void) 함수를 보자.

```c
void vTaskStartScheduler( void )
{
    BaseType_t xReturn;

    traceENTER_vTaskStartScheduler();

	   ...

    if( xReturn == pdPASS )
    {
		
        #ifdef FREERTOS_TASKS_C_ADDITIONS_INIT
        {
            freertos_tasks_c_additions_init();
        }
        #endif

        /* Interrupts are turned off here, to ensure a tick does not occur
         * before or during the call to xPortStartScheduler().  The stacks of
         * the created tasks contain a status word with interrupts switched on
         * so interrupts will automatically get re-enabled when the first task
         * starts to run. */
        portDISABLE_INTERRUPTS();

        #if ( configUSE_C_RUNTIME_TLS_SUPPORT == 1 )
        {
            /* Switch C-Runtime's TLS Block to point to the TLS
             * block specific to the task that will run first. */
            configSET_TLS_BLOCK( pxCurrentTCB->xTLSBlock );
        }
        #endif

        xNextTaskUnblockTime = portMAX_DELAY;
        xSchedulerRunning = pdTRUE;
        xTickCount = ( TickType_t ) configINITIAL_TICK_COUNT;

				...

        ( void ) xPortStartScheduler();

				...
}
```

코드를 보면, 스케줄러를 실행하기 전에 무언가 준비하는 과정임을 알 수 있다.

중간에 보면 portDISABLE_INTERRUPTS(); 가 있는데, 스케줄러는 time tick에 따라 task들을 관리해야 하므로 첫 태스크가 실행되기 전, 즉 스케줄러가 systick을 통해 인터럽트를 발생시켜 태스크를 관리하기 전까지 인터럽트가 발생되지 않도록 disable 상태로 바꿔둔다.

그리고 이 disable 상태는 스케줄러가 태스크를 작동시키기 전까지 풀리지 않는다!

그리고 다시 prvPortStartFirstTask를 보면..

![2](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/3d37a8b5-b613-43b7-a4ab-d7cc7272b3a2)

svc 0에서 hard fault가 발생하니, 여기에 break를 걸어서 레지스터 상태를 살펴보자.

![3](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/c46b0eaf-eeb3-4d35-b67a-46eb6a7392aa)

자. 여기서 basepri 라는 레지스터를 보자.

해당 레지스터는 priority의 base를 설정하는 레지스터인데, 이 레지스터에 저장된 값보다 우선순위가 낮은 Exception들이 활성화 되지 않도록 마스킹처리를 해준다.

아까 전에 스케줄러가 동작하기 전까지 interrupt를 disable 시킨다고 했는데, 이 disable을 수행하는 동작 또한 이 레지스터를 사용한다!

![4](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/ec07360f-2979-4f56-811e-6696fe50fe50)

portmacro.h에 위와 같이 정의되어 있고, vPortRaiseBASEPRI()에 들어가보면,

![5](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/9456eb51-6d05-4791-a3b2-a3917f4e4b17)

이렇게 basepri에 값을 설정하여 인터럽트들을 마스킹한다.

그런데, 우리는 지금 scv 0을 호출하고 있다.

prvPortStartFirstTask에서 svc handler를 호출하기 위해서는 baspri가 0이어야 하는데(그래야 인터럽트들이 호출될 수 있음), 레지스터 상태를 보면 basepri가 80으로 설정된 것을 볼 수 있다.

그래서 svc call 자체가 block 처리되었고,

**결국, svc handler로 진입하기도 전에 뻗어버린 것이다.**

![6](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/4c717dc5-4726-4789-8f6c-b90162c9ad9a)

그래서 svc 0 호출 전, basepri를 0으로 설정해주고 돌리면, 

![7](https://github.com/Kimbongsik/Kimbongsik.github.io/assets/63995044/b5d7798e-6e01-4584-b443-883f4505db2d)

문제없이 잘 동작한다.

사실 STM32CubeIDE에는 이미 FreeRTOS가 들어 있고, 여기에서는 아마 이 문제가 발생하지 않을 것이다. 나는 FreeRTOS github에서 소스코드를 내려받아서 보드에 직접 포팅했기 때문에 FreeRTOS 측에서 해당 문제를 해결해주지 않은 것 같다.

보통 SVC 콜은 우선순위가 높게 할당되어야 하는데, 칩 문제인지 모르겠지만 특정 보드에서 이런 문제가 발생하는듯?
