---
title: "[Web] HTTP / HTTPS"
last_modified_at: 2023-02-16T16:20:02-05:00
categories:
  - web

tags:
  - web
---

## HTTP

http: hyper text transfer protocol

클라이언트가 서버에게 요청하면 서버가 응답함. 웹 서버는 http 서버를 http 서비스 포트에 대기시킴. 포트: TCP/80 , TCP/8080

네트워크 포트 : 서버와 클라이언트가 정보를 교환하는 추상화된 장소. 클라이언트가 서버의 포트에 접근하여 데이터를 내려놓고, 서버가 클라이언트에게 보낼 데이터를 실어서 돌려보냄. (통상 포트)

서비스 포트 : 네트워크 포트 중에서 특정 서비스가 점유하고 있는 포트. ex) http가 80번 포트를 점유하고 있따면 http의 서비스 포트는 80번 포트임. 포트로 데이터를 교환하는 방식은 전송 계층(transport layer)의 프로토콜(TCP, UDP)을 따름. TCP로 데이터를 전송하려는 서비스에 UDP 클라이언트가 접근하면 데이터가 교환되지 않음. 그래서 서비스 포트를 표기할 때는 서비스가 사용하는 전송 계층 프로토콜을 같이 표기함. 예를 들어 http의 서비스 포트가 tcp/80 이라고 하면 http 서비스를 80번 포트에서 tcp로 제공하고 있다는 뜻임. 포트의 개수는 os에서 정하기 나름. 

포트 0번~1023 : well known port, privileged port

ex) 22번 포트: ssh / 80번 포트: http / 443번 포트: https

well known port에 서비스를 실행하려면 관리자 권한이 필요함. 따라서 클라이언트는 이 대역에서 실행 중인 서비스들은 관리자의 것이라고 신뢰할 수 있음.

### Example of Requset

GET

/index.html

HTTP/1.1
Host: [dreamhack.io](http://dreamhack.io/)
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36

### Example of Response

HTTP/1.1

200 OK

Server: Apache/2.4.29 (Ubuntu)

Content-Length: 61

Connection: Keep-Alive

Content-Type: text/html

`<!doctype html>
<html>
<head>
</head>
<body>
</body>
</html>`

## HTTP 메시지

HTTP 메시지에는 클라이언트가 전송하는 http 요청, 그리고 서버가 반환하는 http 응답이 있음.

- http 헤드 - 헤드의 각 줄은 CRLF로 구분되며, 첫 줄은 start-line, 나머지는 헤더라고 부름. 헤드의 끝은 crlf 한 줄로 나타냄. 헤더는 필드와 값으로 구성되며 http 메시지 또는 바디의 속성을 나타냄. 하나의 http 메시지에는 0개 이상의 헤더가 있을 수 있음.
- http 바디 - 헤드의 끝을 나타내느 CRLF 뒤 모든 줄을 말함. 전송하려는 데이터는 바디에 담김.

![1](https://user-images.githubusercontent.com/63995044/219376552-fb0331f0-5b05-4a76-ba17-9b523781eea5.png)
## HTTP 요청

HTTP 요청은 서버에게 특정 동작을 요구하는 메시지임. 서버는 해당 동작이 실현 가능한지, 클라이언트가 그러한 동작을 요청할 권한이 있는지 등을 검토하고, 적절할 때만 이를 처리함.

- 시작 줄 - method, request-url, http version으로 구성. 띄어쓰기로 구별.

**method**는 URI가 가리키는 리소스를 대상으로 서버가 수행하길 바라는 동작을 나타냄. 

ex) GET, POST

**GET : 리소스를 가져와라.** 클라이언트가 서버에 접속하려면 새로운 페이지를 렌더링하기 위해 리소스가 필요함. 이때 브라우저는 GET 요청을 서버에 전송하여 리소스를 받아옴. 

**POST : 리소스로 데이터를 보내라.** 클라이언트에서 서버로 리소스를 생성, 업데이트할 때 쓰이는 메소드(ex: 게시판에 글 쓰기)

- 헤더와 바디

## HTTP 응답

요청에 대한 결과를 반환. 상태정보(status)와 클라이언트에게 전송할 리소스가 응답에 포함됨. HTTP 응답의 시작 줄은 HTTP 버전(서버에서 사용하는 HTTP 프로토콜 버전), status code(ex: 200), reason phrase(상태 코드가 발생한 이유 기술)로 구성되고 각각은 띄어쓰기로 구분됨. 

- status code : 200 _ 성공/ 302 _ 다른 url로 갈 것 / 403 _ 클라이언트가 요청권한이 없음 / 404 _ 리소스가 없음 / 503 _ 서버가 과부하로 인해 요청을 처리할 수 없음

## HTTPS(HTTP over Secure socket layer)

HTTP의 응답과 요청은 평문으로 전달됨. 이를 가로챈다면 중요한 정보가 유출될 수 있는데, 예를 들어 로그인할 때 전송한 POST 요청에는 이용자의 ID와 비밀번호가 포함됨.

→ TLS(Transport Layer Security) 프로토콜을 도입하여 이런 문제점을 보완함. TLS는 서버와 클라이언트 사이에 오가는 모든 HTTP 메시지를 암호화 함. 공격자가 중간에 메시지를 탈취하더라도 이를 해석하는 것은 불가능하고, 결과적으로 HTTP 통신이 도청과 변조로부터 보호됨.

이런 식으로..

![2](https://user-images.githubusercontent.com/63995044/219376557-dda0ed68-2d81-4a8f-8e4d-90df1f4735c4.png)

</br>
반면, http는..

![3](https://user-images.githubusercontent.com/63995044/219376562-c8b9ae8e-8205-46fb-a11e-f46cd9d0d67e.png)