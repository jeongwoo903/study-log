# [HTTP] http의 특징

## 선 개념
- [[HTTP] hypertext와 hypermedia](https://github.com/jeongwoo903/study-log/blob/main/2024-08/%5BHTTP%5D%20hypertext%EC%99%80%20hypermedia.md)

## http 이란?
HyperText Transfer Protocol의 약자로 OSI 7계층 중 application layer의 속해있다.

이름에서 Protocol이란 용어가 붙듯이, Client와 Server간의 정보를 요청(request)하고 응답(response)하기 위한 규약이다.

초기에는 HTML과 같은 hypermedia 문서를 주로 전송했지만, 현재는 Plain text, JSON, XML등 hypertext가 아닌 다양한 형태의 정보고 전송하는 프로토콜이다.

## http의 특징
### Client-Server Architecture
> 리소스를 사용하는 Client와 리소스가 존재하는 Server를 분리시키는 모델

request-reponse 구조로 Client는 Server에 요청을 보내고 Server는 요청에 대해 응답한다.

### 무상태(Stateless) 프로토콜
각 요청은 독립적으로 처리되며, 응답에 대한 결과를 반환하긴 하지만 요청에 대한 정보를 기억하지 않는다.
즉, stateless는 응답에 대한 상태를 기억하지 않는다는 말이다.

쉽게 얘기 하자면,   
어떤 사용자가 admin 인증 요청을 서버에 보내고 admin이 된 채로 사이트를 돌아다녀도 Server는 그 유저가 admin이라는 상태를 저장하고 있지 않다는 말이다.  
admin 임을 서버가 파악하기 위해서는 어떠한 요청을 할 때마가 admin이라는 상태를 담아서 요청을 보내야 한다.

#### 장점
응답에 대한 상태를 기억하지 않기 때문에 서버가 2대하면 아무 서버에서나 응답을 처리해서 넘겨줘도 되기 때문에 **서버의 확장성이 용이하다는 장점을 지닌다.**
또한, 상태를 유지해서 관리하지 않으니까 프로토콜의 구현이 상대적으로 단순해진다.
서버가 고장나도 A 서버에서 하던 일을 B 서버에서도 할 수 있기 때문에 대체성이 좋다.

#### 단점
하지만 예시에서 말했듯이 상태를 저장하지 않아 **매 요청마다 상태를 전달해줘야 한다는 단점**도 있다.  
이러한 점 때문에 로그인과 같이 **인증이 유지되어야 하는 경우 쿠키나 세션, 토큰등을 함께 써주어야 한다.**

### 비연결성(Connectionless) 프로토콜
http는 통신은 한 사이클이 request-response로 이루어진다.
즉, 한 사이클이 돌면 연결 상태를 유지할 필요가 없음으로 서버가 응답을 하면 연결은 종료된다.

#### 장점
한 사이클이 끝나면 연결을 종료하기 때문에 서버의 자원을 효율적으로 관리 할 수 있다.

#### 단점
연결이 끊기면 요청 시 TCP/IP 연결을 새로 맺어야 하기 때문에 3 way handshake 시간이 추가된다.
또한 웹브라우저 요청시 HTML, CSS, JS, 이미지 등 여러 자원이 함께 다운로드 되어 시간이 오래 걸린다.
