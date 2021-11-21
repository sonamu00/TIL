# IP(인터넷 프로토콜)

- 지정한  IP주소에 데이터 전달하기 위한 규칙
- 패킷(Packet)이라는 통신 단위로 데이터 전달

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a0ab67a6-a704-4882-9dd3-a5938b52b019/http1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a0ab67a6-a704-4882-9dd3-a5938b52b019/http1.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/efa190ad-9521-4172-9e9d-d5fd518cbb08/http2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/efa190ad-9521-4172-9e9d-d5fd518cbb08/http2.png)

- 출발 지점(클라이언트)IP와 목적 지점(서버)IP, 기타 정보를 가진 것이 **패킷**

## IP 프로토콜의 한계

- 비연결성
    - 패킷을 받을 대상이 없거나 서비스 불능 상태에서도 패킷 전송
        
        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26033c36-7b48-460a-a8b8-1a7a7712ce72/http3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26033c36-7b48-460a-a8b8-1a7a7712ce72/http3.png)
        
- 비신뢰성
    - 중간에 패킷이 사라질 수 있음
    - 패킷이 순서대로 오지 않을 수 있음(데이터가 클 경우 나눠서 옴)
        
        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9abb155-86a8-4bc4-ade5-c6c80043b206/HTTP4.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9abb155-86a8-4bc4-ade5-c6c80043b206/HTTP4.png)
        
        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70a761d2-c3ca-4405-b1e9-f8ff70976769/HTTP5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70a761d2-c3ca-4405-b1e9-f8ff70976769/HTTP5.png)
        
- 프로그램 구분
    - 같은 IP를 사용하는 서버에서 통신하는 다른 애플리케이션들을 구분하지 못함