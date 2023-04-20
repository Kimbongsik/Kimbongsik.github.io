---
title: "[네트워크] DHCP(Dynamic Host configuration Protocol), IP 주소 자동 할당 설정 방법, IoT 제어 방법"
last_modified_at: 2023-04-20T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - network

tags:
  - network
---

## DHCP

- IP 주소 및 기타 통신 매개변수를 네트워크에 연결된 장치에 자동으로 할당
- 무선으로 연결된 경우 DHCP를 많이 사용
- 대부분의 가정용 네트워크에서는 라우터가 IP 주소를 장치에 할당하는 DHCP 서버의 역할을 수행
- 서버, 라우터가 DHCP 서버 역할을 할 수 있음
- 라우터가 DHCP 사용할 경우
    - 제외시킬 IP 범위 정의
    - IP, subnetmask
    - DNS IP
    - Default gateway IP
    - Domain gateway IP
    - Domain-Name
    - 임대기간
    - 서버 실행
    

## 라우터 ↔ 서버 간 사용

라우터와 서버 간에서는 DHCP를 절대 사용하지 않는다. 라우터와 서버 사이 스위치에 여러 엔드 디바이스들이 연결돼 있는 경우, 라우터가 재부팅 되면 내부 IP가 재할당 된다. 이 때 IP 주소가 변경될 수 있으므로 엔드 디바이스의 외부 통신이 불가능해진다. 서버 또한 재부팅 되어 IP 주소가 변경된다면 당연히 서버의 IP를 모르기 때문에 서비스를 받을 수 없는 문제가 발생한다.

즉, DHCP는 엔드 디바이스와 라우터 간 사용되는 것이지, 라우터↔서버 사이에서 쓰는 프로토콜이 아니다. 

## 라우터 IP 설정

Router> en

Router#conf t

Router(config)# int gi0/0/0

Router(config)# ip add 163.180.116.1 255.255.255.0

Router(config-if)#no shutdown

# 라우터가 DHCP 서버일 경우

## DHCP 설정

`Router>en`

`Router#conf t`

- router(config) 모드 접속

`Router(config)#ip dhcp excluded-address 163.180.116.1`

- 할당 가능한 첫번째 주소는 라우터에 주로 할당 되므로, ~.1 제외
- ~.0의 경우, 라우터에서 자동으로 할당해주지 않음

`Router(config)#ip dhcp excluded-address 163.180.116.255`

- broadcast 주소 제외

`Router(config)#ip dhcp pool test`

- DHCP 서버 이름 설정

`Router(dhcp-config)#network 163.180.116.0 255.255.255.0` 

- DHCP를 통해 할당되는 IP 주소 대역 및 서브넷 마스크 설정

`Router(dhcp-config)#dns-server 1.1.1.1`

- DNS 서버 설정

`Router(dhcp-config)#default-router 163.180.116.1`

- default gateway 설정

`Router(dhcp-config)#exit`

`Router(config)#service dhcp`

- 서비스 실행

## 실행 화면

![1](https://user-images.githubusercontent.com/63995044/233310289-8b32ab60-c460-4c22-ad60-0ef778b86edf.png)

![2](https://user-images.githubusercontent.com/63995044/233310294-3cb8c966-57e8-4e34-aef4-f0cd5243005a.png)

DHCP로 정상적으로 IP를 받아오는 걸 볼 수 있다.

# 서버가 DHCP 서버인 경우

위에서 언급했듯, 라우터는 DHCP로 할당해주면 안 된다.

위의 방법대로 라우터의 IP를 알아서 설정해주고,

## 서버 설정

>서버 Static IP 설정

![3](https://user-images.githubusercontent.com/63995044/233310297-86122316-986d-4407-9779-5ec7c6e3c3cf.png)

>DHCP 설정

- Start IP Address
    - 203.230.7.1(라우터 IP)과 203.230.7.2(서버 IP)는 제외해줘야 함
    - 대략 +100번지 정도 해준다

![4](https://user-images.githubusercontent.com/63995044/233310299-559f643e-88ff-40f2-a448-4b077bc7e01b.png)

## 실행 화면

![5](https://user-images.githubusercontent.com/63995044/233310302-b96fd7c5-3bf1-4fb8-8684-32cf6046d215.png)

![6](https://user-images.githubusercontent.com/63995044/233310303-8242b4d7-64c5-4933-80e8-145698988acc.png)

PC를 눌러 확인했을 때 정상적으로 DHCP IP를 받아오는 것을 확인할 수 있다.

</br>

# 네트워크 외부에 DHCP 서버를 두는 경우

그런데, 위의 경우 네트워크의 개수가 많아지면 각 네트워크 영역마다 DHCP를 일일이 지정해줘야 한다는 불편함이 생긴다. 이런 경우에 외부에 DHCP 서버를 두고 IP를 할당해주면 편하다. 아래와 같이 수행.

![7](https://user-images.githubusercontent.com/63995044/233310306-1722dc97-0db8-4d02-aae2-912dccbc0c89.png)

![8](https://user-images.githubusercontent.com/63995044/233310310-d7ce8f93-8305-485d-b7be-088cea6037d1.png)

</br>

그리고 DHCP로 잘 받아오는지 PC2에서 확인해보면..!

</br>

![9](https://user-images.githubusercontent.com/63995044/233310315-d6a6b30c-a105-437b-a089-4db035309612.png)

안 된다. 

## 라우터 설정

현재 서버에서는 7.~ 대역의 네트워크에 IP를 할당할 수 있는 내용이 저장되어 있다고 하더라도, PC에서 서버에 리퀘스트를 요청할 때, 연결되어 있는 라우터가 처리하게 된다. 이 때 라우터는 서로 다른 네트워크에 대한 요청을 전달해주지 않는다.

그래서 라우터 설정을 따로 해줘야 한다. 

`Router(config)#int gi0/0/0`

- Request가 네트워크 ~7.x 대역에서 오므로, gi 0/0/0에 대해 설정해줘야 한다

`Router(config-if)#ip helper-address 203.230.8.2`

- ip helper-address [DHCP 서버 주소] 입력

## 실행 화면

![10](https://user-images.githubusercontent.com/63995044/233310318-19d15638-1f28-4946-b34c-6f2912708deb.png)

짠~ 잘 받아오는 것을 확인할 수 있다. 

해당 기능을 라우터에 구현할 수도 있긴 하지만, 잘 쓰지 않는다. 라우터는 라우팅 하기도 바쁘기 때문에.. DHCP 서버로서 역할을 수행하게 되면 리소스가 부족한 상황이 발생할 수도 있다. 

# IoT 디바이스 IP 자동 할당(DHCP)

이제 위에서 배운 내용을 바탕으로, IoT 기기에 IP를 DHCP를 사용해 자동으로 할당해보자. 아래와 같이 구성한다. 

서버 장치에서 DHCP service ON, IoT service ON 해주고, PC에 접속해 웹브라우저로 들어가자.

![11](https://user-images.githubusercontent.com/63995044/233310320-9e50b97f-7a83-4cef-b09d-ee6f55ecd204.png)

Sign up 해준 다음, IoT 디바이스 설정을 해준다.

(Motion Detector와 Webcam은 무선랜을 쓰는 게 default 이므로, Advanced 설정에 들어가 Fast Ethernet(FE) 으로 바꿔주자)

### IoT 기기 설정 방법

Config > DHCP 선택

Config > Settings > IoT Server > Remote Server 선택

Server Address: 203.230.7.1

User Name : admin

Password : admin

→ Connect

</br>

그럼 PC에서 서버에 접속했을 때 다음과 같이 디바이스들이 뜬다.

![12](https://user-images.githubusercontent.com/63995044/233310323-648a7ed4-026f-4bf0-ad34-0bbeff6b4881.png)