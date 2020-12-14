### 서비스(일반)

- 생산된 재화를 운반, 배급, 제공하는 것

### 서비스(컴퓨터)

- 다수의 사용자가 필요한 정보를 제공하는 것
- 사용자(클라이언트)가 요청한 정보르 컴퓨터(서버)가 응답하여 제공하는 것
    - 사용자 역할 프로그램 → 클라이언트(Client) 프로그램
    - 서버 역할 프로그램 → 서버(Server) 프로그램
- 다양한 종류의 정보가 필요할 수 있음 → 필효한 정보의 종류에 따라 서비스의 종류가 달라짐

### WEB의 구성요소

- WEB system : WEB client(browser), WEB server, WAS(Web Application Server), Database 등등
- WEB language : html, css, sss, sql 등등
- WEB protocol : HTTP

### WEB Client(WEB Browser)

- WEB Service를 위한 사용자 인터페이스를 제공
- 사용자가 지정한 정보로 WEB 요청 데이터(HTTP Requset)를 생성하고 전달받은 응답 데이터(HTTP Response)를 파싱하여 화면에 출력함
    - 파싱(parsing) : 데이터를 분석하여 원하는 형태로 추출하는 행위

### WEB Server

- WEB Client가 전달한 요청(HTTP Request)을 해석하여 사용자가 요구하는 웹 페이지(HTML data)를 응답(HTTP Response)하는 시스템
- 정적인 요청에 대한 요청만 처리할 수 있음(HTML 요청)
- 동적인 요청이 전달되면 WAS에게 처리를 맡기고 처리 결과에 대해서 HTTP응답을 생성함(SSS 요청)

### WAS(Web Application Server)

- 사용자가 전달하는 데이터를 서버 측에서 처리하기 위해 개발한 서버 프로그램
- 사용자가 전달한 값을 기반으로 프로그램의 동작이 달라짐(동적 서비스)
- 정적 웹 서버 프로그램과 데이터베이스 사이에서 상호 연동 역할을 수행함
    - 사용자의 입력 값 → 정적 서버가 추출 후 WAS에 전달 → WAS의 명령 코드에 따라 Database에 전달됨 → Database의 처리결과가 WAS에 전달됨 → 처리 결과에 맞는 응답 데이터 구성 → 결과를 사용자에게 전달함

### HTML(Hyper Text Markup Language)

- WEB data 생성을 위해 사용되는 **HyperText**를 구성하는 마크업 언어
    - Hyper Text
        - Hyper Link(특정 정보 시작지점 표식)를 통해 구성된 다른 정보로 쉽게 연결시켜주는 구조화된 문서
        - node : 정보(웹 페이지)
        - link : node를 연결하는 고리
        - Depth : 상위 node에서 현재 node까지의 경로 수
        - jump : 선택에 따라 링크를 통해 다음 node로 이동
    - 마크업 언어
        - 태그 등을 이용하여 문서나 데이터의 구조를 정의하는 언어
    - 태그
        - 특정 데이터의 의미, 값을 표현하기 위한 식별자

### CSS(Client Side Script)

- Client에서 웹 페이지를 동적으로 표현하기 위해 사용되는 스크립트 언어
- Client에서 처리되는 명령이므로 서버가 source code 원형을 전달함
- 한번 parsing 동작을 거친 data에 재접근하여 추가 동작을 지원함

### SSS(Server Side Script)

- Server 프로그램의 동작을 동적으로 처리하기 위해 사용되는 스크립트 언어
- WAS에서 사용되는 언어
- 클라이언트로부터 전달되는 데이터의 처리, 데이터베이스로 연동 되는 명령 등을 처리하기 위해 사용됨
- Server에서 처리되는 명령이므로 처리 결과만 Client에게 반환함

### HTTP(Hyper Text Transfer Protocol)

- 인터넷에서 데이터를 주고 받을 수 있는 프로토콜(데이터를 주고 받기 위한 규칙)
- 데이터를 주고 받기 위한 각각의 데이터 요청이 독립적으로 관리됨
- TCP/IP 네트워크 상에서 Server/Client 기반으로 동작 함
