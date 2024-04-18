---
title: "[네트워크] TCP/IP Protocol Suite"
last_modified_at: 2024-04-18T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - network

tags:
  - network
---
# TCP/IP 프로토콜 그룹

현재 인터넷에서는 5개의 계층으로 구성된 계층적 모델을 채택하고 사용하고 있다.

![TCPIP_layer](https://user-images.githubusercontent.com/63995044/133731054-928db249-f5e3-45b4-87fa-f9a5fd2e97f5.png)

> 물리층 :
- 프레임의 bit를 다음 링크로 전달 책임  
- 전송 매체 이용
- 전기 또는 광학 신호 전송

> 데이터 링크층 :
- 유무선 링크를 통해 frame 전달 책임
- 상위층으로부터 datagram을 받아 프레임으로 캡슐화 
- 다양한 링크층 프로토콜에 따라 서로 다른 서비스 제공

> 네트워크 층 :
- 발신지 컴퓨터와 목적지 컴퓨터간 연결 생성을 책임
- host-대-host 통신
- 가능한 경로를 통해 패킷을 라우팅하기 위한 책임 담당
- 흐름제어, 오류제어, 혼잡제어 서비스를 제공하지 않는 비연결형 프로토콜
- IP, ICMP, IGMP, DHCP, ARP  

> 전송층 :
- 논리적 연결은 종단-대-종단
- 상위층(응용층)으로부터 메시지를 받아 전송층 패킷으로 캡슐화(segment 또는 data)하여 목적지 호스트의 전송층에 전달 책임
- 응용층에 서비스 제공
- TCP(전송 제어 프로토콜): 연결형-흐름제어, 오류제어, 혼잡제어 제공 , UDP(사용자 데이터그램 프로토콜): 비연결형 , SCTP

> 응용층 :
- 논리적 연결은 종단-대-종단
- 서로 응용층 간 메시지 교환
- 프로세스간 통신 제공
- HTTP, sMTP, FTP, TELNET, SSH, SNMP, DNS

연결구조가 다음과 같이 A로부터 B로 Communication 하는 구조를 가지고 있을 때,  


###                                              A(Sender) ----Link 1-------Router------Link2---- B(Reciever)


Source host와 Destination host 간에 송신하는 통신 프로토콜, 즉 규칙이 동일해야 한다.  
Source(A)와 Destination(B) 양 호스트는 물리층, 데이터 링크층, 네트워크층, 전송층, 응용층 5개의 모든 계층과 관련이 있다.
Link1과 Link2 스위치의 경우에는 데이터 링크층과 물리층, 두 개의 계층과 관련이 있으며, Router는 여기에 네트워크층을 더해 총 3개의 계층과 관련이 되어 있다.  
응용층, 전송층, 네트워크층 의무는 종단-대-종단 이며, 데이터링크층과 물리층의 의무는 홉(hop)-대-홉이다. 여기서 홉(hop)은 호스트와 라우터이며, 스위치는 홉에 속하지 않는다.   
프로토콜 계층화에서 각 장치에 있는 각 계층은 동일한 객체(object)를 가진다.  


Application-Application : messages  
Transport-Transport : segment or user datagram  
Network-Network : datagram  
Data link-Data link : frame  
Physical-Physical : bits  