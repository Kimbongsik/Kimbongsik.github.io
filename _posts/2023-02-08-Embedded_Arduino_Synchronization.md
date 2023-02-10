---
title: "[아두이노] 동기화(Semaphore), 키패드&모터 제어"
last_modified_at: 2023-02-08T16:20:02-05:00
categories:
  - embedded

tags:
  - embedded
---

아두이노를 사용해 태스크 간 동기화 실습을 진행해본다.

> 요구사항
> 
- Keypad ISR과 KeyPad Task의 동기화
    - Keypad Task에서는 Keypad ISR로부터의 버튼 전달을 기다림
    - Keypad ISR에서 눌린 버튼을 전역변수를 통해 키패드에 전달
    - 동기화를 위해 바이너리 세마포어 사용
- Keypad Task와 FND Task
    - FND Task에서는 키패드 Task로부터의 버튼 전달을 기다림
    - KeyPad Task는 눌린 버튼을 전역 변수를 통해 FND Task에 전달
    - FND Task는 전달 받은 버튼을 FND에 왼쪽으로 shift하며 표시

## 실습 1

```cpp
#include <FreeRTOS_AVR.h>

#define MS2TICKS(ms) (ms/portTICK_PERIOD_MS)
#define FND_SIZE 6

const int Keypad[4] = {2,3,21,20};
const int FndSelectPin[6] = {22,23,24,25,26,27};
const int FndPin[8] = {30,31,32,33,34,35,36,37};
const int FndFont[10] = {0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x60, 0x7D, 0x07, 0x7F, 0x67};

SemaphoreHandle_t Sem;
int SendValue = 0;
int Fnd[FND_SIZE] = {0,};

//FND의 값을 왼쪽으로 shift
//파라미터로 들어온 새로운 값(data)을 0번지에 넣는 함수

void ShiftInsert(int data){
  int i;
  for(i=1; i<FND_SIZE; i++){
    Fnd[FND_SIZE - i] = Fnd[FND_SIZE - i -1];
  }
  Fnd[0] = data;
}

void KeypadControl1(){
  delay(50);
  SendValue = 1;
  xSemaphoreGive(Sem);
}

void KeypadControl2(){
  delay(50);
  SendValue = 2;
  xSemaphoreGive(Sem);
}

void KeypadControl3(){
  delay(50);
  SendValue = 3;
  xSemaphoreGive(Sem);
}

void KeypadControl4(){
  delay(50);
  SendValue = 4;
  xSemaphoreGive(Sem);
}

void KeypadTask(void* arg){
  int i;
  int keypad;

  while(1){
    //세마포어를 통해 키패드가 눌렸음을 Keypad ISR로부터 전달 받음
    //portMAX_DELAY: 세마포어를 받기 전까지 대기
    //입력된 번호는 ShiftInsert를 통해 Fnd[]에 삽입
		//세마포어를 받을 때까지 대기
    if(xSemaphoreTake(Sem,portMAX_DELAY)){
      keypad = SendValue;
      ShiftInsert(keypad);
    }
  }
}

void FndSelect(int pos){
  int i;

  for(i=0; i<6; i++){
    if(i==pos){
      digitalWrite(FndSelectPin[i], LOW);
    }
    else{
      digitalWrite(FndSelectPin[i], HIGH);
    }
  }
}

void FndDisplay(int pos, int num){
  int i;
  int flag = 0;
  int shift = 0x01;

  FndSelect(pos);

  for(i=0; i<8; i++){
    flag = (FndFont[num] & shift);
    digitalWrite(FndPin[i], flag);
    shift <<= 1;
  }
}

//지속적으로 Fnd 데이터를 FND에 출력 
void FndTask(void* arg){
  int i;
  while(1){
    for(i=0; i<FND_SIZE; i++){
      delay(3);
      FndDisplay(i,Fnd[i]);
    }
  }
}

void setup(){
  int i;
  for(i=0; i<6; i++){
    pinMode(FndSelectPin[i], OUTPUT);
  }
  for(i=0; i<8; i++){
    pinMode(FndPin[i], OUTPUT);
  }
  for(i=0; i<4; i++){
    pinMode(Keypad[i], INPUT);
  }

  attachInterrupt(0,KeypadControl1, RISING);
  attachInterrupt(1,KeypadControl2, RISING);
  attachInterrupt(2,KeypadControl3, RISING);
  attachInterrupt(3, KeypadControl4, RISING);

  vSemaphoreCreateBinary(Sem);

  xTaskCreate(KeypadTask, NULL, 200, NULL, 2, NULL);
  xTaskCreate(FndTask, NULL, 200, NULL, 1, NULL);
  vTaskStartScheduler();
}

void loop(){
  
}
```

> 요구사항
> 
- Keypad를 통해 Motor 제어
    - 왼쪽 회전, 정지, 오른족 회전 3개 키에 의해 모터 동작
    - Keypad ISR과 Motor Task간 전역변수와 세마포어를 이용해 데이터 전달

## 실습2

```cpp
#include <FreeRTOS_AVR.h>

const int MT_P = 10;
const int MT_N = 9;
const int LeftKey = 2;
const int StopKey = 3;
const int RightKey = 21;

QueueHandle_t xQueue;
SemaphoreHandle_t Sem;
int SendValue = 0;

//SendValue로 값 전달, 세마포어로 동기화
void LeftKeyControl() {
  SendValue = 1;
  xSemaphoreGive(Sem);
}
void StopKeyControl() {
  SendValue = 2;
  xSemaphoreGive(Sem);
}
void RightKeyControl() {
  SendValue = 3;
  xSemaphoreGive(Sem);
}

void MotorTask(void * arg) {

  while(1) {
    //세마포어를 통해 키패드가 눌렸음을 KeypadISR로부터 전달 받았다면, 아래 구문 실행
    if(xSemaphoreTake(Sem, portMAX_DELAY)){
    // Rotate left
    if(SendValue == 1) {
    digitalWrite(MT_P, LOW);
    digitalWrite(MT_N, HIGH);
  }
  // Stop
  else if(SendValue == 2) {
    digitalWrite(MT_P, LOW);
    digitalWrite(MT_N, LOW);
  }
  // Rotate right
  else if(SendValue == 3) {
    digitalWrite(MT_P, HIGH);
    digitalWrite(MT_N, LOW);
    }
   }
  }
}

void setup() {
  pinMode(MT_P, OUTPUT);
  pinMode(MT_N, OUTPUT);
  pinMode(LeftKey, INPUT);
  pinMode(StopKey, INPUT);
  pinMode(RightKey, INPUT);

  attachInterrupt(0, LeftKeyControl, RISING);
  attachInterrupt(1, StopKeyControl, RISING);
  attachInterrupt(2, RightKeyControl, RISING);

  vSemaphoreCreateBinary(Sem);

  xTaskCreate(MotorTask,NULL,200,NULL,1,NULL);
  vTaskStartScheduler();
}

void loop() {
}
```

> 요구사항
> 
- Keypad를 통해 Motor 제어
    - KeyPad ISR과 KeyPad Task간 전역변수와 semaphore를 이용해 데이터 전달
    - KeyPad Task에서는 Circular Queue에 버튼을 입력
        - 이때 debounce 해결
    - Motor Task에서는 Circular Queue로부터 버튼을 입력받아 모터 제어
    - KeyPad Task(producer)와 Motor Task(consumer)간critical section 보호 및 동기화를 위해 semaphore를 사용
    - 운영체제 Producer & Consumer의 bounded buffer problem 참조

## 실습3

```cpp
#include <FreeRTOS_AVR.h>

const int MT_P = 10;
const int MT_N = 9;
const int LeftKey = 2;
const int StopKey = 3;
const int RightKey = 21;

int Queue[8] = {0,};
int in = 0;
int out = 0;
int cur = 0; // 디바운싱을 위한 변수 선언

SemaphoreHandle_t Sem;
SemaphoreHandle_t Sem2;
int SendValue = 0;

//SendValue로 값 전달, 세마포어로 동기화
void LeftKeyControl() {
  SendValue = 1;
  xSemaphoreGive(Sem);
}
void StopKeyControl() {
  SendValue = 2;
  xSemaphoreGive(Sem);
}
void RightKeyControl() {
  SendValue = 3;
  xSemaphoreGive(Sem);
}

void KeypadTask(void* arg){

  while(1){
    vTaskDelay(50);
    
    if(SendValue == cur){
      continue;
    }
    if(xSemaphoreTake(Sem,portMAX_DELAY)){
        //큐에 데이터를 넣음(Producer)
        //Serial.print(SendValue);
          Queue[in] = SendValue;
          in = (in+1) % 8;
          cur = SendValue;
          xSemaphoreGive(Sem2);
          Serial.print("SendValue");
          Serial.println(SendValue);
          Serial.print("cur");
          Serial.print(cur);
    }
  }
}

void MotorTask(void * arg) {

  while(1) {
    vTaskDelay(10);
    if(xSemaphoreTake(Sem2,portMAX_DELAY)){
    // Rotate left
    if(Queue[out] == 1) {
    digitalWrite(MT_P, LOW);
    digitalWrite(MT_N, HIGH);
    }
    // Stop
    else if(Queue[out] == 2) {
      digitalWrite(MT_P, LOW);
      digitalWrite(MT_N, LOW);
    }
    // Rotate right
    else if(Queue[out] == 3) {
      digitalWrite(MT_P, HIGH);
      digitalWrite(MT_N, LOW);
    }
    out = (out+1) % 8;
   }
  }
}

void setup() {
  Serial.begin(9600);
  pinMode(MT_P, OUTPUT);
  pinMode(MT_N, OUTPUT);
  pinMode(LeftKey, INPUT);
  pinMode(StopKey, INPUT);
  pinMode(RightKey, INPUT);

  attachInterrupt(0, LeftKeyControl, RISING);
  attachInterrupt(1, StopKeyControl, RISING);
  attachInterrupt(2, RightKeyControl, RISING);

  //vSemaphoreCreateBinary(Sem);
  //vSemaphoreCreateBinary(Sem2);
  Sem = xSemaphoreCreateCounting(8,0);
  Sem2 = xSemaphoreCreateCounting(8,0);

  if(Queue != NULL){
    xTaskCreate(MotorTask,NULL,200,NULL,1,NULL);
    xTaskCreate(KeypadTask,NULL,200,NULL,2,NULL);
    vTaskStartScheduler();
  }
  
}

void loop() {
}
```
