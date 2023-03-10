---
title: "[보안] SSH란"
last_modified_at: 2023-03-13T16:20:02-05:00
categories:
  - security

tags:
  - security
---

# SSH(Secure Shell Protocol)

 네트워크 프로토콜 중 하나로 컴퓨터와 컴퓨터가 인터넷과 같은 Public Network를 통해 서로 통신을 할 때 보안적으로 안전하게 통신을 하기 위해 사용하는 프로토콜임.

- 데이터 전송
- 원격 제어

이 두 가지 경우에 많이 사용한다. 

데이터 전송의 예로는 원격 저장소인 ‘깃허브’가 있음. 소스 코드를 원격 저장소인 깃허브에 푸쉬할 때 SSH를 활용해 파일을 전송하게 됨. 

원격 제어의 예로는 AWS의 인스턴스 서버에 접속하여 해당 머신에 명령을 내리기 위해서도 SSH를 통한 접속을 해야함. 

그럼 FTP 같은 통신을 위해 사용되는 프로토콜도 있는데 SSH를 사용하는가를 생각해볼 수 있다. 

그 이유는 ‘보안’이다. FTP를 통해 민감한 정보를 주고 받으면 정보를 직접 네트워크를 통해 넘기기 때문에 누구나 해당 정보를 열어볼 수 있어 보안에 취약하다.

SSH는 다른 컴퓨터와 통신을 하기 위해 접속을 할 때 일반적으로 사용하는 비밀번호의 입력을 통한 접속을 하지 않음. 기본적으로 한 쌍의 키, Private Key, Public Key 를 통해 접속하려는 컴퓨터와 인증 과정을 거침.

Public Key는 공개되어도 비교적 안전한 키로, 이를 통해 메시지를 전송하기 전 암호화를 함. 그러나 복호화는 불가능. Private Key는 절대 외부에 노출되어서 안 되는 키로 본인의 컴퓨터 내부에 저장하게 되어 있는데 이를 통해 암호화된 메시지를 복호화 할 수 있음. 

<통신 방법>

1. Public Key를 통신하고자 하는 컴퓨터에 복사하여 저장함. 
2. 요청을 보내는 클라이언트 사이드 컴퓨터에서 접속 요청을 할 때 응답을 하는 서버 사이드 컴퓨터에 복사되어 저장된 Public key와 클라이언트 사이드에 해당 Public Key와 쌍을 이루는 Private Key와비교를 하여 서로 한 쌍의 키인지 아닌지를 검사함. 
3. 서로 관계를 맺고 있는 키라는 것이 증명되면 두 컴퓨터 사이에 암호화된 채널이 형성되어 키를 활용해 메시지를 암호화하고 복호화하며 데이터를 주고 받을 수 있게 됨.
