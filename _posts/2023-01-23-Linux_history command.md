---
title: "[리눅스/linux] history 명령어 및 원리"
last_modified_at: 2023-01-23T16:20:02-05:00
categories:
  - linux

tags:
  - 리눅스
---

리눅스 기반 OS에서 쉘 커맨드를 이용하다 보면 같은 명령어를 반복적으로 타이핑 할 필요 없이, 방향키를 통해 간단하게 이전 명령어들을 불러올 수 있다. 이건 시스템을 재부팅 시켜도 없어지지 않는데, 이 명령어들이 어디에 저장되며, 라이프 타임은 언제인지 궁금해져서 찾아봤다.


## history 명령어

history는 GNU에서 지원하는 라이브러리다.

![1](https://user-images.githubusercontent.com/63995044/214060667-64eae1e6-402f-48e5-8937-d598d332a13b.png)

history는 이전에 사용했던 명령어들의 리스트들을 파일에 저장해두었다가 보여준다. man 명령어를 통해 확인해보면 bash 셸에서 지원한다는 것을 확인할 수 있었다. history는 유저별로 자신의 히스토리 파일을 가지고 있고, 홈 디렉토리에 숨김 파일로 위치해 있다. 

![2](https://user-images.githubusercontent.com/63995044/214063544-4a2751e4-c3fb-401f-88cd-d68badbc6d70.png)


 cat으로 확인해보면 이전에 쳤던 명령어들이 나열되어 있는 것을 볼 수 있다.

![3](https://user-images.githubusercontent.com/63995044/214063679-0a602c6c-20fc-4add-8aa4-5203eb386723.png)

- HISTFILE : command 저장파일
- HISTFILESIZE : 히스토리 파일 최대 크기
- HISTSIZE : 히스토리에 저장 가능한 최대 명령어 개수


## 원리

1. 로그인 해서 bash 터미널에 들어가면 파일의 내용을 로드
2. RAM에 현재 active 되어 있는 shell의 명령어 이력들을 저장할 메모리 공간을 만듦(=history list)
3. shell에 입력해 실행하는 모든 커맨드들은 파일에 저장되는 것이 아닌 history list에 저장
4. 램에 저장된 명령어들은 bash shell이 종료될 때 ‘~./bash_history’파일에 추가됨

(→ 파일에 있는 값을 가져와서 history list에 불러오기 때문에, 재부팅 후 history를 쳐도 이력을 확인할 수 있는 것)

<aside>
💡 참고 사이트: [https://jhnyang.tistory.com/306]
</aside>