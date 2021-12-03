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

# HTTP 기본

## HTTP 역사

- HTTP/0.9 1991년: GET 메서드만 지원, HTTP 헤더X
- HTTP/1.0 1996년: 메서드, 헤더 추가
- **HTTP/1.1 1997년**: 가장 많이 사용, 우리에게 가장 중요한 버전
    - RFC2068 (1997) -> RFC2616 (1999) -> RFC7230~7235 (2014)
- HTTP/2 2015년: 성능 개선
- HTTP/3 진행중: TCP 대신에 UDP 사용, 성능 개선

## 기반 프로토콜

- TCP: HTTP1.1,  HTTP/2
- UDP: HTTP/3
- 현재 HTTP/1.1 주로 사용
    - HTTP/2, HTTP/3도 점점 증가

## HTTP 특징

- 클라이언트 서버 구조
- 무상태 프로토콜(stateless), 비연결성
- HTTP 메세지
- 단순함, 확장 가능

## 클라이언트 서버 구조

- Request Response 구조
- 클라이언트는 서버에 요청을 보내고, 서버의 응답을 대기
- 서버가 요청에 대한 결과를 만들어서 응답

![](img/061.png)

## 무상태 프로토콜(Stateless)

- 서버가 클라이언트 상태를 보존 X
- 장점: 서버 확장성이 높음(스케일 아웃)
- 단점: 클라이언트가 추가 데이터 전송

## Stateful, Stateless 차이점

- Stateful(상태 유지)
    - 중간에 다른 서버로 바뀌면 안됨(바뀔 경우 다른 서버에 상태 정보를 알려줘야함)
    - 예) 로그인 세션 유지
    
    ![](img/062.png)
    
    ![](img/063.png)
    
- Stateless(무상태)
    - 중간에 다른 서버로 바꿔도 됨
    - 갑자기 클라이언트 요청이 증가해도 서버를 대거 투입 가능
    - 응답 서버를 쉽게 바꿀 수 있다 → **무한한 서버 증설 가능**
    - 예) 로그인이 필요없는 단순한 서비스 소개 화면
    
    ![](img/064.png)
    
    ![](img/065.png)
    
    ![](img/066.png)
    

## 비 연결성

![](img/067.png)

![](img/068.png)

- HTTP는 기본이 연결을 유지하지 않는 모델
- 일반적으로 초 단위의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이
하로 매우 작음
- 예) 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않는다.
- 서버 자원을 매우 효율적으로 사용할 수 있음

## 비 연결성 한계와 극복

- TCP/IP 연결을 새로 맺어야 함 - 3 way handshake 시간 추가
- 웹 사이트로 요청하면 css, 자바스크립트, 추가 이미지 등 수많은 자원이 함께 다운로드
- 지금은 HTTP 지속연결로 문제 해결
- HTTP/2, HTTP/3 에서 더 많은 최적화

![](img/069.png)

![](img/0610.png)

## HTTP 메세지

![](img/0611.png)

### 시작 라인 - 요청 메세지

![](img/0613.png)

- HTTP 메서드(GET 조회)
    - 종류: GET(리소스 조회), POST(요청 내역 처리), PUT, DELETE...
    - 서버가 수행해야 할 동작 지정
- 요청 대상 (/search?q=hello&hl=ko)
    - absolute-path[?query] (절대경로[?쿼리])
    - 절대경로: "/"로 시작하는 경로
    - 참고: *, http://...?x=y 와 같이 다른 유형의 경로지정 방법도 있다.
- HTTP 버전
    - HTTP/1.1, HTTP/3 같은 버전을 나타냄

### 시작 라인 - 응답 메세지

![](img/0612.png)

- HTTP 버전
- HTTP 상태 코드: 요청 성공, 실패를 나타냄
    - 200: 성공
    - 400: 클라이언트 요청 오류
    - 500: 서버 내부 오류
- 이유 문구: 사람이 이해할 수 있는 짧은 상태 코드 설명 글

## HTTP 헤더

![](img/0614.png)

- HTTP에 필요한 모든 부가정보
- 예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트(브라우저) 정보,
서버 애플리케이션 정보, 캐시 관리 정보...

## HTTP 바디

![](img/0615.png)

- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능

# HTTP 메서드

## API URI 설계

- 가장 중요한 것은 **리소스 식별**
- URI의 계층 구조를 활용해서 설계
- 리소스와 행위를 분리한다
    - 예시
        - 회원 목록 조회: /read-member-list →  /members
        - 회원 조회: /read-member-by-id →  /members/{id}
        - 회원 등록: /create-member → /members/{id}
        - 참고: 계층 구조상 상위를 컬렉션으로 보고 복수단어 사용 권장(member → memebers)
    - 리소스: 회원
    - 행위: 조회, 등록, 수정, 삭제

## HTTP 메서드 종류

- 주요 메서드
    - GET: 리소스 조회
    - POST: **요청 데이터 처리**, 주로 등록에 사
    - PUT: 리소스를 대체, 해당 리소스가 없으면 생성
    - PATCH: 리소스 부분 변경
    - DELETE: 리소스 삭제
- 기타 메서드
    - HEAD: GET과 동일하지만 메세지 부분을 제외하고, 상태 줄과 헤더만 반환
    - OPTIONS: 대상 리소스에 대한 통신 가능 옵션(메서드)을 설명(주로 CORS에서 사용)
    - CONNECT: 대상 자원으로 식별되는 서버에 터널을 설정
    - TRACE: 대상 리소스에 대한 경로를 따라 메세지 루프백 테스트를 수행

## GET

- 리소스 조회
- 서버에 전달하고 싶은 데이터는 query를 통해서 전달
- 메세지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는 곳이 많아서 권장하지 않음

## POST

- 요청 데이터 처리
- **메세지 바디를 통해 서버로 요청 데이터 전달**
- 서버는 요청 데이터를 처리
    - 메세지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행함
- 주로 전달된 데이터로 신규 리소스 등록, 프로세스 처리에 사용

![](img/POST1.png)

![](img/POST2.png)

![](img/POST3.png)

- 리소스 URI에 POST 요청이 오면 **데이터를 어떻게 처리할지** 리소스마다 따로 정해야함
    
    ⇒ **정해진 것이 없음**
    

### POST 정리

1. 새 리소스 생성(등록)
    - 서버가 아직 식별하지 않은 새 리소스 생성
2. 요청 데이터 처리
    - 단순히 데이터를 생성하거나, 변경하는 것을 넘어서 프로세스를 처리해야 하는 경우
        - 예) 주문에서 결제완료 → 배달시작 → 배달완료 처럼 값 변경을 넘어서 프로세스의 상태가 변경되는 경우
    - POST의 결과로 새로운 리소스가 생성되지 않을 수도 있음
        - 예) POST/orders/{orderId}/start-delivery
            
            ⇒ 리소스만으로 URI를 구성할 수 없는 경우에 **컨트롤 URI**를 사용함
            
3. 다른 메서드로 처리하기 애매한 경우
    - 예) JSON으로 조회 데이터를 넘겨야하는데, GET메서드를 사용하기 어려운 경우
    - 애매하면 POST

## PUT

- 리소스를 대체
    - 리소스가 있으면 대체
    - 리소스가 없으면 생성
    - 그냥 덮어쓰기 기능
- **클라이언트가 리소스를 식별**
    - 클라이언트가 리소스 위치를 알고 URI 지정
    - POST와 큰 차이점 ⇒ POST는 리소스 위치를 모른다
- 주의점: 리소스를 완전히 대체하기 때문에 하나의 필드만 수정하고 다른 필드를 명시하지 않으면  다른 필드들은 삭제된다.
    
![](img/PUT3.png)
    

![](img/PUT4.png)

![](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd58a11f-cd25-4ab5-bd7d-b3dca480a84d/PUT1.png)

![](img/PUT2.png)

## PATCH

- 리소스 부분 변경
- PUT과 달리 하나의 필드만 변경할 때 다른 필드가 없어도 다른 필드들은 그대로 유지된다.

![](img/PATCH.png)

## DELETE

- 리소스 제거

![](img/DELETE2.png)

![](img/DELETE.png)

- 캐시가능(Cacheable Methods)
    - 응답 결과 리소스를 캐시해서 사용해도 되는가?
    - GET, HEAD, POST, PATCH 캐시 가능
    - 실제로는 GET, HEAD 정도만 캐시로 사용
        - POST, PATCH는 본문 내용(BODY)까지 캐시 키로 고려해야 하는데, 구현이 쉽지 않음

# HTTP 메서드 활용
## 클라이언트에서 서버로 데이터 전송

### 데이터 전달 방식

- 쿼리 파라미터를 통한 데이터 전송
    - GET
    - 주로 정렬 필터(검색어)
- 메세지 바디를 통한 데이터 전송
    - POST, PUT, PATCH
    - 회원가입, 상품주문, 리소스 등록, 리소스 변경

### 데이터를 전달하는 4가지 상황

- 정적 데이터 조회
    - 이미지, 정적 텍스트 문서
- 동적 데이터 조회
    - 주로 검색, 게시판 목록에서 정렬 필터(검색어)
- HTML Form을 통한 데이터 전송
    - 회원 가입, 상품 주문, 데이터 변경
- HTML API를 통한 데이터 전송
    - 회원 가입, 상품 주문, 데이터 변경
    - 서버 to 서버, 앱 클라이언트, 웹 클라이언트(Ajax)

## 정적 데이터 조회

- 이미지 정적 텍스트 문서
- 조회는 GET 사용
- 정적 데이터는 일반적으로 쿼리 파라미터 없이 리소스 경로로 단순하게 조회함

![](img/method1.png)

## 동적 데이터 조회

- 주로 검색, 게시판 목록에서 정렬 필터(검색어)
- 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용
- 조회는 GET를 사용
- GET은 쿼리 파라미터 사용해서 데이터 전달

![](img/METHOD2.png)

## HTML Form 데이터 전송

![](img/METHOD3.png)

- HTML Form Submit시 POST 전송
    - 예) 회원 가입, 상품 주문, 데이터 변경
- Content-Type: application/x-www-form-urlencoded 사용
    - form의 내용을 메세지 바디를 통해 전송(key=value, 쿼리 파라미터 형식)
    - 전송 데이터를 url encoding 처리
        - 예) abc김 -> abc%EA%B9%80
        

![](img/METHOD4.png)

- HTML Form은 GET 전송도 가능
    
    

![](img/METHOD5.png)

- Content-Type: multipart/form-data
    - 파일 업로드 같은 바이너리 데이터(이진 데이터) 전송시 사용
    - 다른 종류의 여러 파일과 폼의 내용 함께 전송
- 참고: HTML Form 전송은 GET, POST만 지원

## HTML API 데이터 전송

- 서버 to 서버
    - 백엔드 시스템 통신
- 앱 클라이언트
    - 아이폰, 안드로이드
- 웹 클라이언트
    - HTML Form 전송 대신 자바 스크립트를 통한 통신에 사용(AJAX)
    - 예) React, VueJs 같은 웹 클라이언트와 API 통신
- POST, PUT, PATCH: 메세지 바디를 통해 데이터 전송
- GET: 조회, 쿼리 파라미터로 데이터 전달
- Content-Type: application/json을 주로 사용 (사실상 표준)
    - TEXT, XML, JSON 등등

![](img/METHOD6.png)

## HTTP API 설계

- HTTP API - 컬렉션
    - POST 기반 등록
    - 예) 회원 관리 API 제공
- HTTP API - 스토어
    - PUT 기반 등록
    - 예) 정적 컨텐츠 관리, 원격 파일 관리
- HTML FORM 사용
    - 웹 페이지 회원 관리
    - GET, POST만 지원

### 회원 관리 시스템 - 컬렉션

- 예시
    - 회원 목록 /members -> GET
    - 회원 등록 /members -> POST
    - 회원 조회 /members/{id} -> GET
    - 회원 수정 /members/{id} -> PATCH, PUT, POST
    - 회원 삭제 /members/{id} -> DELETE
- 신규 자원 등록 특징(POST)
    - **클라이언트는 등록될 리소스의 URI를 모른다**
    - **서버**가 새로 등록될 리소스 URI를 생성해준다
        - HTTP/1.1 201 Created
        Location: **/members/100**
    - 컬렉션(Collection)
        - 서버가 관리하는 리소스 디렉토리를 의미
        - **서버가 리소스의 URI의                                                                                                                                                                                                                                                                                    리소스를 생성하고 관리**
        - 여기서 컬렉션은 /members
    
    ### 파일 관리 시스템 - PUT 기반 설계
    
    - 예시
        - 파일 목록 / les -> GET
        - 파일 조회 / les/{ lename} -> GET
        - 파일 등록 / les/{ lename} -> PUT
        - 파일 삭제 / les/{ lename} -> DELETE
        - 파일 대량 등록 / les -> POST
    - 신규 자원 등록 특징(PUT)
        - **클라이언트가 등록될 리소스의 URI를 알고 있어야 한다**
            - 파일 등록 / files/{filename} -> PUT
            - PUT/**files/star.jpg**
        - 클라이언트가 직접 리소스의 URI를 지정한다
        - 스토어(Store)
            - 클라이언트가 관리하는 리소스 저장소
            - 클라이언트가 리소스의 URI를 알고 관리
            - 여기서 스토어는 /files
    
    ### HTML FORM 사용
    
    - 예시
        - 회원 목록 /members → GET
        - 회원 등록 폼 /members/**new** → GET
        - 회원 등록 /members/**new**, /members → POST
        - 회원 조회 /members/{id} → GET
        - 회원 수정 폼 /members/{id}/**edit** → GET
        - 회원 수정 /members/{id}/**edit**, /members/{id} → POST
        - 회원 삭제 /members/{id}/**delete** → POST
    - HTML FORM은 GET, POST만 지원
    - 컨트롤 URI
        - GET, POST만 지원하므로 제약이 있음
        - 이런 제약을 해결하기 위해 동사로 된 리소스 경로 활용
        - POST의 /new, /edit, /delete가 컨트롤 URI
        - HTTP 메서드로 해결하기 애매한 경우 사용(HTTP API 포함)
    
    ## 참고하면 좋은 URI 설계 개념
    
    - 문서(document)
        - 단일 개념(파일 하나, 객체 인스턴스, 데이터베이스 row)
        - 예) /members/100, / les/star.jpg
    - 컬렉션(collection)
        - 서버가 관리하는 리소스 디렉터리
        - 서버가 리소스의 URI를 생성하고 관리
        - 예) /members
    - 스토어(store)
        - 클라이언트가 관리하는 자원 저장소
        - 클라이언트가 리소스의 URI를 알고 관리
        - 예) / les
    - 컨트롤러(controller), 컨트롤 URI
        - 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스 실행
        - 동사를 직접 사용
        - 예) /members/{id}/delete
    
    [REST API 공식문서](https://restfulapi.net/resource-naming)