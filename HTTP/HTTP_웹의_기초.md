# HTTP:웹의 기초
- [HTTP:웹의 기초](#http웹의-기초)
  - [HTTP 개론](#http-개론)
  - [URL와 리소스](#url와-리소스)
    - [URL(Uniform Resource Locator)](#urluniform-resource-locator)
    - [URL 문법](#url-문법)
    - [Scheme 종류](#scheme-종류)
  - [HTTP 메시지](#http-메시지)
## HTTP 개론
- http: 신뢰성있는 전송 프로토콜
- 미디어 타입
  - MIME(Multipupose Internet mail Extensions)
    - 다른 전자 메일 시스템 사이 통신을 지원하기위해 설계
    - 웹 서버가 객체 데이터를 전송하기 위해서 사용됨
- URI(Uniform Resource Identifier)
  - URL(Uniform Resource Locator): 리소스의 구체적 위치
  - URN(Uniform Resource Name): 리소스의 위치와 상관없이 사용되는 고유한 이름
- 트랜잭션
  - (요청 + 응답)으로구성
- TCP/IP
  - HTTP는 신뢰성을 TCP/IP에 의존
  - TCP는 오류없는 전송, 순서, 조각나지않는 데이터 스트림 보장
- 프로토콜 버전
  - HTTP/0.9    : GET 메소드만 지원
  - HTTP/1.0    : 버전 번호, 헤더, 추가 메서드 멀티미디어 객체 추가
  - HTTP/1.0+   : "keep-alive" 커넥션, 가상 호스팅, 프록시 연결
  - HTTP/1.1    : 성능 최적화
  - HTTP/2.0    : 구글의 SPDY 프로토콜을 기반으로 설계
## URL와 리소스
### URL(Uniform Resource Locator)
- 인터넷의 리소스를 가리키는 표준 이름
- URL은 {sheme}://{address}/{resource}의 형태를 가진다.
  - http://www.google.com/index.html 을 요청한다고 가정했을 때
  - http://는 scheme을 의미하며 http 프로토콜을 사용함을 나타낸다.
  - www.google.com은 서버의 위치를 의미한다.
  - /index.html은 요청하고자 하는 리소스의 경로를 의미한다.
### URL 문법
URL을 scheme에 따라서 다른 문법을 가진다. URL 문법을 정규화하면 

> \<scheme\>://\<username\>:\<password\>@\<host\>:\<port\>/\<path\>;\<parameter\>?\<query\>#\<fragment\>

이렇게 나타낼 수 있다.

| 구성요소  | 설명 | 기본값 |
|-------- |-----                              |------|
|scheme   | 리소스에 접급할 때 사용하는 프로토콜을 의미 | 없음 |
|username | 특정 스킴에서 사용하는 사용자 이름 | anonymous|
|password | 사용자의 비밀번호| 이메일 주소|
|host     | 서버의 호스트명 또는 IP 주소 | 없음 |
|port     | 서버가 열어놓은 포트 번호, 스킴에 따라 기본 포트가 다르다 | 스킴에 따름 |
|path     | 서버 내의 리소스의 위치 | 없음|
|parameter| 특정 스킴에서 사용하는 파라미너 | 없음|
|query    | 스킴에서 질의 요소를 작성하는데 쓰임| 없음|
|fragment | 리소스의 조각이나 일부분을 가리킴 | 없음 |

### Scheme 종류
- http: Hypertext Transfer protocol
- https: http 커넥션 양 끝단에서 암호화하기 위한 보안 소켓 계층(Security Sockets Layer, SSL)이 있다.
- mailto: 이메일 전송을 위함 스킴
- ftp: File Transfer Protocol로 파일 요청을 위해서 사용된다. 
- rtsp, rtspu: Real Time Streaming Protocol로 오디오 및 비디오와 같은 미디어 리소스 전달을 위해 사용된다. rtspu는 UDP를 사용함을 나타낸다.
- file: 호스트 기기에 바로 접근할 수 있는 파일을 나타낸다. 
- news: 특정 문서나 뉴스 그룹에 접근하는데 사용된다. 
- telnet: 대화형 서비스에 접근하는데 사용된다. 

## HTTP 메시지