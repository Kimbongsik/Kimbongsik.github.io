---
title: "[Web] 웹 브라우저"
last_modified_at: 2023-01-31T16:20:02-05:00
categories:
  - web

tags:
  - web
---


## URL

이용자가 주소창에 dreamhack.io를 입력했다. 

1. dreamhack.io를 해석한다. 
2. dreamhack.io에 해당하는 주소를 탐색한다. (DNS 요청)
3. HTTP를 통해 dreamhack.io에 요청한다.
4. dreamhack.io의 HTTP 응답을 수신한다.
5. 리소스를 다운로드 하고 웹 렌더링(HTML, CSS, JavaScript)한다.

URL(Uniform Resource Locator)의 약자로, 웹에 있는 리소스의 위치를 표현하는 문자열. 브라우저로 특정 웹 리소스에 접근할 때, URL을 사용하여 이를 서버에게 요청함. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/43357f49-52b0-433d-ba89-6e94aac1ad79/Untitled.png)

- Scheme : 웹 서버와 어떤 프로토콜로 통신할 것인가?
- Host : 접속할 웹 서버의 주소에 대한 정보
- Port: 접속할 웹 서버의 포트에 대한 정보
- Path : 접근할 웹 서버의 리소스 경로
- Query : 웹 서버에 전달하는 파라미터로 URL에서 ‘?’ 뒤에 위치함.
- Fragment : 메인 리소스에 존재하는 서브 리소스를 접근할 때 이를 식별하기 위한 정보를 담고 있음. # 문자 뒤에 위치함.

## Domain Name

Host는 웹 브라우저가 접속할 웹 서버의 주소를 나타내는데, Host는 Domain Name, IP address의 값을 가질 수 있음. IP는 외우기 어려우므로 domain name을 ip 대신 사용함. 

domain name을 host 값으로 이용할 때 브라우저는 DNS(Domain Name Server)에 Domain Name을 질의하고, DNS가 응답한 IP Address를 사용함. 예를 들어 웹 브라우저에서 [http://ex.com에](http://ex.com에) 접속할 경우, DNS에 질의해 얻은 ex.com의 IP와 통신함. 

결론: DNS : Host의 도메인 이름을 IP로 변환하거나 IP를 도메인 이름으로 변환해주는 서버

## 웹 렌더링

서버로부터 받은 리소스를 이용자에게 시각화 하는 행위. 

서버의 응답을 받은 웹 브라우저는 리소스의 타입을 확인하고, 적절한 방식으로 이용자에게 전달함. 서버로부터 HTML과 CSS를 받으면 HTML을 파싱하고 CSS를 적용하여 이용자에게 보여줌. 웹 렌더링은 웹 렌더링 엔진에 의해서 이뤄지는데, 브라우저별로 서로 다른 엔진을 사용함. 사파리는 Webkit, Blink, Gecko 엔진을 사용함.


참고 사이트: dreamhack
