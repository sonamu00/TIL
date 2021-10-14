#  REST
분산 하이퍼미디어 시스템(ex 웹)을 위한 아키텍쳐 스타일(제약조건의 집합)

## REST 구성
### 자원(RESOURCE) - URI
- URI는 정보의 자원을 표현해야 한다.
- 리소스명은 동사 대신 명사를 사용해야 한다.
  - ex) GET/members/delete/1 -> 행위에 대한 표현 X

### 행위(Verb)
- 자원에 대한 행위는 HTTP Method를 사용한다.
- GET, POST, PUT, DELETE 와 같은 메서드를 제공한다.
  - ex) DELETE/members/1

### 표현(Representations)
- 자원은 어떤 형태로 데이터를 주고 받는지 정해야 한다.
  - ex) Content-Type: JSON, XML, TEXT 등..

## REST를 구성하는 스타일
### Clinent - Server 구조
- 클라이언트 응용 프로그램과 서버 응용 프로그램이 서로에 대한 종속성 없이 별도로 발전할 수 있어야 함을 의미한다. 
- 클라이언트는 리소스 URI만 알아야 한다. 
- 서버와 클라이언트는 독립적으로 개발될 수 있다.

### Stateless(무상태성)
- 서버는 클라이언트가 보낸 HTTP 요청을 저장하지 않는다.
- 세션 정보나 쿠키 정보를 별도로 저장하고 관리하지 않기 때문에 API 서버는 들어오는 요청만을 단순히 처리하면 된다.
- 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않음으로써 구현이 단순해진다.

### Cacheable(캐시 가능)
- REST는 HTTP 웹 표준을 그대로 사용하기 때문에 HTTP가 가진 캐싱 기능을 적용할 수 있다.
- HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하면 캐싱 구현이 가능하다.
  
### Uniform interface
- identification of resources
  - 리소스가 URI로 식별되야 한다.
- manipulation of resources through representations
  - representation 전송을 통해 리소스를 조작해야 한다.
  - ex) 리소스를 생성, 수정, 삭제를 할 때 HTTP 메세지에 표현을 담아서 전송
- self-descriptive messages 
  - REST API 메시지만 보고도 이를 쉽게 이해 할 수 있는 자체 표현 구조로 되어 있어야 한다.
- hypermedia as the engine of application state (HATEOAS)
  - 애플리케이션의 상태가 Hyperlink를 통해 전이되어야 한다.

self-descriptive messages와 HATEOAS는 대부분의 REST API들이 지키지 못하고 있다.

### Layered system(계층형 구조)
- 서버 A에 API를 배포하고 서버 B에 데이터를 저장하고 서버 C에서 요청을 인증하는 계층화된 시스템 아키텍처를 사용할 수 있다.
- 클라이언트는 일반적으로 종단 서버에 직접 연결되어 있는지 아니면 중간에 연결되어 있는지 알 수 없다.
-  보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고 PROXY, 게이트웨이 같은 네트워크 기반의 중간매체를 사용할 수 있게 한다.

### Code-on-demand(optional)
- 서버에서 코드를 클라이언트로 보내서 실행할 수 있어야 한다.
  - ex) JavaScript

## self-descriptive messages
HTTP 메세지만 보고서 무슨 뜻인지 해석이 되어야 하는데 대부분의 REST API들은 그렇지 않다. 아래 예시를 보도록 하자.

```HTTP
GET / HTTP/1.1
```
목적지가 빠져 있어서 Self-descriptive 하지 않다.

```HTTP
GET / HTTP/1.1
Host: www.example.org
```
위와 같이 목적지를 명시해야 한다.

```HTTP
HTTP/1.1 200 OK
[ { "op": "remove", "path": "/a/b/c" } ]
```
어떤 문법으로 작성되었는지 클라이언트가 해석할 수 없기 때문에 Self-descriptive 하지 않다.

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json
[ { "op": "remove", "path": "/a/b/c" } ]
```
이렇게 하면 클라이언트가 Content-Type을 보고 해석할 수 있으나 여전히 Self-descriptive 하지 않다. 왜냐하면 "op", "path"가 어떤 뜻인지 한 번에 알 수 없기 때문이다.

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json-patch+json

[ { "op": "remove", "path": "/a/b/c" } ]
```
json-patch+json 처럼 미디어 타입으로 정의 되어 있는 메세지를 사용한다. 이와 같이 작성하면 json-patch라는 명세를 보고 메세지를 해석할 수 있다. 비로소 완전한 Self-descriptive가 완성된다.

## HATEOAS
애플리케이션 상태가 Hyperlink로 전의되어야 한다.
```http
HTTP/1.1 200 OK
Content-Type : text/html

<html>
<head></head>
<body><a href="/test">test</a></body>
</html>
```
html은 a태크의 하이퍼링크를 통하여 상태 전이가 가능하다. 

```http
HTTP/1.1 200 OK
Content-Type: application/json
Link: </article/1>; rel="previous", </article/3>; rel="next";

{
	"title": "The second article",
	"contents": "blah"
}
```
json은 Link라는 헤더를 통해 다른 리소스를 가리킬 수 있다.


## Reference
- https://www.youtube.com/watch?v=RP_f5dMoHFc
- https://restfulapi.net/
- http://www.incodom.kr/REST
- https://woojinger.tistory.com/32