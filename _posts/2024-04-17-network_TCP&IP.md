---
title: "[네트워크] TCP/IP란"
last_modified_at: 2024-04-17T16:20:02-05:00
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - network

tags:
  - network
---

# TCP/IP란    
+ TCP/IP : 컴퓨터-컴퓨터간의 LAN(지역 네트워크), WAN(광역 네트워크)에서 통신을 하기 위한 프로토콜.
미국 APRANET(아파넷)에서 시작되어 미국 방위 통신청에서 통신을 위해 TCP/IP를 사용하게 한 것이 시초.     
* IP: 인터넷에 연결되어 있는 장치 단말기, 컴퓨터 사이에서 데이터 패킷을 전송하기 위해 만든 일종의 주소 같은 개념이다.
178.145.57.144 와 같은 형태로 만들어진다. 
* TCP: Transmission Control Protocol. 서버와 클라이언트 사이에서 신뢰성 있게 데이터를 주고 받기 위해 만들어진 프로트콜.    
네트워크 선로에서 데이터가 전달 될 때 발생하는 데이터 손실을 잡아내어 교정 및 재조합 과정을 거친다. 데이터를 전송하기 전에 연결을 만드는 연결지향 프로토콜.     

# Server Socket/Client Socket
+ 클라이언트 프로그램과 서버 프로그램은 각각 포트를 통해 연결, 데이터 교환이 이루어진다. 포트를 사용하기 위해 소켓을 사용해야 하는데, 
소켓은 응용 프로그램과 소켓 사이의 인터페이스 역할을 한다. 통신을 위한 통로 같은 느낌이다. Node.js에서는 소켓을 사용하여 IP주소와 포트를 사용하여 
서로 다른 시스템이 통신이 가능하다. 동일 서버에서 실행 중인 IPC(Inter-Process Communication)에도 사용 가능하다. 여기서 소켓은 서버, 클라이언트로 존재한다. 
서버는 연결을 수신하고, 클라이언트는 서버에 연결을 진행한다. 

* 클라이언트.js
```javascript
var net = require('net'); //net 모듈 사용
//8000번 포트로 접속 
var socket = net.connect({port:8000, host:'localhost'}, function(){
    console.log('Client connected');
    setInterval(function(){
        socket.write('hi');
        }, 1000); //1000ms 간격으로 hi 서버로 요청
 });
 //서버로 받은 데이터 출력
 socket.on('data', function(data){
     console.log(data.toString());
 });
 //종료 시 Client disconnect 메시지 출력
 socket.on('end', function(){
     console.log('Client disconnected');
 });
 socket.on('error', function(err){
     console.log(err);
 });
 ```
 
 * 서버.js
 ```javascript
var net = require('net');

var server = net. createServer(function(socket){
   console.log(socket.address().address + "connected.");
    
   socket.on('data',function(data){
       console.log(data);
   });
   socket.on('close',function(){
       console.log('client disconnected');
   });
   //클라이언트 접속 시 Welcome 화면에 출력
   socket.wirte('Welcome');
});
//에러 메시지
server.on('error', function(err){
   console.log('Err'+ err);
});

//8000포트로 접속 대기
server.listen(5000, function(){
   console.log('listening on 8000..');
});