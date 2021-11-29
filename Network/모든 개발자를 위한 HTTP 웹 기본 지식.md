# IP(인터넷 프로토콜)

- 지정한  IP주소에 데이터 전달하기 위한 규칙
- 패킷(Packet)이라는 통신 단위로 데이터 전달

![http1](img/http1.png)

![http2](img/http2.png)

- 출발 지점(클라이언트)IP와 목적 지점(서버)IP, 기타 정보를 가진 것이 **패킷**

## IP 프로토콜의 한계

#### 비연결성
- 패킷을 받을 대상이 없거나 서비스 불능 상태에서도 패킷 전송
        
        ![http3](img/http3.png)
        
#### 비신뢰성
- 중간에 패킷이 사라질 수 있음
- 패킷이 순서대로 오지 않을 수 있음(데이터가 클 경우 나눠서 옴)
        
![http4](img/HTTP4.png)
        
![http5](img/HTTP5.png)
        
#### 프로그램 구분
- 같은 IP를 사용하는 서버에서 통신하는 다른 애플리케이션들을 구분하지 못함

# TCP와 UDP

## 인터넷 프로토콜의 4계층

- 애플리케이션 계층 - HTTP, FTP
- 전송 계층 - TCP, UDP
- 인터넷 계층 - IP
- 네트워크 인터페이스 계층

![](img/HTTP6.png)

⇒ 단계마다 하나에 패킷에 각각의 패킷 정보을 추가함

## TCP 특징(전송 제어 프로토콜)

#### 연결지향 - TCP 3 way handshake(가상 연결)

![](img/TCP1.png)

#### 데이터 전달 보증

![](img/TCP2.png)

#### 순서 보장

![](img/http7.png)

- 신뢰할 수 있는 프로토콜
- 현재 대부분 사용함
    
    

## TCP 3 way handshake

![](img/http8.png)

- SYN(접속 요청)과 ACK(요청 수락)를 3번 주고 받은 후 데이터를 전송
    - 최근엔 3번에서 데이터 전송이 이루어지기도 함

## UDP 특징(사용자 데이터그램 프로토콜)

- 기능이 거의 없음(하얀 도화지에 비유)
- TCP 3 way handshake X
- 데이터 전달 보증 X
- 순서 보장 X
- 데이터 전달 순서가 보장되지 않지만, 단순하고 빠름
- IP와 거의 같다. ⇒PORT와 체크섬(메세지 검증 데이터)
- 애플리케이션에서 추가 작업 필요

  
# PORT

- 한글로 번역하면 항구 ⇒ 출발과 도착 지점이라고 생각하자
- 같은 IP 내에서 프로세스를 구분/

![](img/PORT1.png)

![](img/PORT2.png)

## PORT 번호

- 0 ~ 65535 할당 가능
- 0 ~ 1023: 잘 알려진 포트, 사용하지 않는 것이 좋음
    - FTP - 20, 21
    - TELNET - 23
    - HTTP - 80
    - HTTPS - 443


# DNS

## IP의 단점

- 기억하기 어렵다
- IP가 변경될 경우 찾아갈 수 없다

![](IMG/DNS1.png)

![](IMG/DNS2.png)

## DNS(도메인 네임 시스템)

- 전화번호부 같은 것
- 도메인 명을 IP주소로 변환
- 목적 지점을 찾아갈 때 도메인 명으로 찾아감

![](IMG/DNS3.png)

# URI
## URI, URL, URN의 차이

- URI는 locator(위치), name 또는 둘 다 추가로 구분될 수 있음
- URI는 URL과 URN를 총괄하는 의미이다(URL과 같은 의미로 쓰기도 함)

![](img/uri1.png)

![](img/URL2.png)

## URI의 단어 뜻

- Uniform: 리소스를 식별하는 통일된 방식
- Resource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
- Identifier: 다른 항목과 구분하는데 필요한 정보

## URL, URN의 단어 뜻

- URL - Locator: 리소스가 있는 위치를 지정
- URN - Name: 리소스에 이름을 부여
- 위치는 변할 수 있지만 이름은 안 변함
- URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음

## URL 문법

- `scheme://[userinfo@]host[:port][/path][?query][#fragment]`
    - https://www.google.com:443/search?q=hello&hl=ko

### scheme

- 주로 프로토콜 사용
    - 프로토콜: 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙
        - 예) http, https, ftp 등
- http는 80포트, https는 443포트를 주로 사용, 포트는 생략 가능
    - https는 http에 보안 추가한 것(HTTP Secure)

### userinfo

- URL에 사용자 정보를 포함해서 인증
- 거의 사용하지 않음

### host

- 호스트명
- 도메인명 또는 IP 주소 사용

### port

- 접속 포트
- 일반적으로 생략함, 생략시 http는 80, https는 443

### path

- 리소스 경로, 계층적 구조
- 파일 디렉토리 구조와 같음
- 예 )
    - /home/file1.jpg
    - /members
    - /members/100, /items/iphone12

### query

- key=value 형태
- ?로 시작, &으로 추가
- query parameter, query string 등으로 불림, 웹서버에 제공하는 파라미터, 문자 형태

### frament

- html 내부 북마크에 사용
- 서버에 전송하는 정보 아님