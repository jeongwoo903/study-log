# [Network] 소켓 제작과 의문

## 제작 과정

### 1. `socket()` 호출
```c
#include <stdio.h>      // printf, perror
#include <stdlib.h>     // exit
#include <sys/socket.h> // socket
#include <unistd.h>     // close

#include <sys/types.h>
#include <netinet/in.h>     // sockaddr_in

int main(int argc, char *argv[]) {
    int sockfd;

    /*
    * 1. socket() 호출
    * AF_INET : 네트워크 주소 체계를 알려준다. 'AF_INET'은 32비트 주소를 사용한다는 뜻이다(IPv4)
    * type : 소켓이 어떤 타입(TCP/UDP 등)인지 알려준다. 'SOCK_STREAM'은 TCP 프로토콜을 사용한 통신 소켓이라는 뜻이다.
    * protocol : 프로토콜 정보; 앞의 두개 인자를 이용해 프로토콜을 특정할 수 없을 때에만 따로 명시해준다. 그 이외에는 예시에서 처럼 0으로 처리해줘도 된다.
    */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    if (sockfd < 0) {
        perror("socket() failed");
        exit(1);
    }

    printf("서버 소켓 생성 테스트 (파일 디스크립터 넘버 = %d)\n", sockfd);

    close(sockfd);

    return 0;
}
```
이는 아주 단순하게 socket을 만드는 과정이다.

현재 이 소켓은 데이터 통신을 하는 소켓은 아니고, 우선 클라이언트의 요청을 기다리고 수락하기 위한 소켓이다.

아래 두 헤더 파일은 필요없어 보이지만 없으면 컴파일 과정에서 에러가 난다.  
```c
#include <sys/types.h>
#include <netinet/in.h> 
```

### 2. `bind()`, IP/포트 연결
소켓을 만들었으면 서버가 사용할 IP 주소와 포트 번호를 생성한 소켓에 결합(bind)시켜야 한다.
c에서는 `bind()`를 이용하면 되는데,

`bind()`는 말 그대로: 
> **"이 소켓은 이 IP주소의 이 포트에서 클라이언트 요청을 기다릴 거야!"** 라고 커널에게 알려주는 단계이다.

```c
#include <stdio.h>      // printf, perror
#include <stdlib.h>     // exit
#include <string.h>     // bzero, memset
#include <sys/socket.h> // socket
#include <unistd.h>     // close
#include <sys/types.h>  // accept
#include <netinet/in.h> // bind

int main(int argc, char *argv[]) {
    int sockfd; // 소켓 디스크립터
    int portno; // 포트 번호

    /* sockaddr_in : 서버 주소 정보를 저장하는 구조체 */
    struct sockaddr_in serv_addr;

    /*
    * [ socket() 호출 ]
    * AF_INET : 네트워크 주소 체계를 알려준다. 'AF_INET'은 32비트 주소를 사용한다는 뜻이다(IPv4)
    * type : 소켓이 어떤 타입(TCP/UDP 등)인지 알려준다. 'SOCK_STREAM'은 TCP 프로토콜을 사용한 통신 소켓이라는 뜻이다.
    * protocol : 프로토콜 정보; 앞의 두개 인자를 이용해 프로토콜을 특정할 수 없을 때에만 따로 명시해준다. 그 이외에는 예시에서 처럼 0으로 처리해줘도 된다.
    */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    /* socket() 호출 실패 시 오류 처리 */
    if (sockfd < 0) {
        perror("socket() - 소켓 생성 실패");
        exit(1);
    }

    /* 
    * [ 서버 주소 구조체 초기화 ]
    * bzero() 함수로 serv_addr 구조체의 모든 바이트 영역을 0으로 초기화한다. (왜? 안하면 가비지 데이터가 들어갈 수도 있어서)
    * 포트 번호는 명령어의 인자로 받아서 정수로 변환한다.
    * sin_family: 주소 체계(IPv4)
    * sin_addr.s_addr: 서버 IP 주소(INADDR_ANY: 모든 IP 주소)
    * sin_port: 서버 포트 번호(바이트 오더 변환)
    */
    memset(&serv_addr, 0, sizeof(serv_addr));
    portno = atoi(argv[1]);
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(portno);
    
    /* 
    * bind() 호출 실패 시 오류 처리
    * bind() 함수는 소켓에 주소를 바인딩하는 함수이다.
    * 소켓 디스크립터, 서버 주소 구조체, 서버 주소 구조체 크기를 인자로 받는다. 
    */
    if (bind(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind() - 주소 바인딩 실패");
        close(sockfd);
        exit(1);
    }

    printf("서버 소켓 생성 테스트 (파일 디스크립터 넘버 = %d)\n", sockfd);

    close(sockfd);

    return 0;
}

```
 `bind()` 까지 하고 나면 이제 가게 문을 열어 두는 것 까지 했다.  
다음은 `listen()`으로 손님 받을 준비를 해야한다.  

`listne()`을 하지 않으면 클라이언트가 아무리 요청을 보내도 서버가 “수신 대기” 상태가 아니라서 거부된다.

### 3. `listen()`, 클라이언트 요청 대기

클라이언트가 서버에 연결 요청을 하면,
그걸 받아들이고 새로운 통신 전용 소켓을 만들어주는 함수이다.

즉, `accept()`는 서버의 문을 열고 손님을 자리에 앉히는 행위!

```c
#include <stdio.h>      // printf, perror
#include <stdlib.h>     // exit
#include <string.h>     // bzero, memset
#include <sys/socket.h> // socket
#include <unistd.h>     // close
#include <sys/types.h>  // accept
#include <netinet/in.h> // bind

#define MAX_CLIENTS 5
#define DEFAULT_PORT 10000

int main(int argc, char *argv[]) {
    /*
    * sockfd : 서버 소켓 디스크립터 (연결 요청을 기다리는 소켓)
    * newsockfd : 새로운 소켓 디스크립터 (연결된 클라이언트와 소통하기 위한 소켓)
    * portno : 포트 번호
    */
    int sockfd, newsockfd;
    int portno = DEFAULT_PORT;

    /* 
    * sockaddr_in : 서버 주소 정보를 저장하는 구조체
    * cli_addr : 클라이언트 주소 정보를 저장하는 구조체
    */
    struct sockaddr_in serv_addr, cli_addr;

    /*
    * [ socket() 호출 ]
    * AF_INET : 네트워크 주소 체계를 알려준다. 'AF_INET'은 32비트 주소를 사용한다는 뜻이다(IPv4)
    * type : 소켓이 어떤 타입(TCP/UDP 등)인지 알려준다. 'SOCK_STREAM'은 TCP 프로토콜을 사용한 통신 소켓이라는 뜻이다.
    * protocol : 프로토콜 정보; 앞의 두개 인자를 이용해 프로토콜을 특정할 수 없을 때에만 따로 명시해준다. 그 이외에는 예시에서 처럼 0으로 처리해줘도 된다.
    */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    /* socket() 호출 실패 시 오류 처리 */
    if (sockfd < 0) {
        perror("socket() - 소켓 생성 실패");
        exit(1);
    }

    /* 
    * [ 서버 주소 구조체 초기화 ]
    * bzero() 함수로 serv_addr 구조체의 모든 바이트 영역을 0으로 초기화한다. (왜? 안하면 가비지 데이터가 들어갈 수도 있어서)
    * 포트 번호는 명령어의 인자로 받아서 정수로 변환한다. 없을 경우 default 포트 10000번을 사용한다.
    * sin_family: 주소 체계(IPv4)
    * sin_addr.s_addr: 서버 IP 주소(INADDR_ANY: 모든 IP 주소)
    * sin_port: 서버 포트 번호(바이트 오더 변환)
    */
    memset(&serv_addr, 0, sizeof(serv_addr));
    if (argc < 2) {
        printf("포트 번호가 지정되지 않음 - 기본 포트 %d번 사용\n", portno);
    } else {
        portno = atoi(argv[1]);
    }
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(portno);
    
    /* 
    * [ bind() 호출 실패 시 오류 처리 ]
    * bind() 함수는 소켓에 주소를 바인딩하는 함수이다.
    * 소켓 디스크립터, 서버 주소 구조체, 서버 주소 구조체 크기를 인자로 받는다. 
    */
    if (bind(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind() - 주소 바인딩 실패");
        close(sockfd);
        exit(1);
    }

    /* 
    * [ listen() 호출 ]
    * listen() 함수는 소켓을 수동 모드로 전환하고, 연결 요청을 대기 상태로 전환한다.
    * 소켓 디스크립터, 연결 요청 대기 큐의 최대 길이를 인자로 받는다.
    */
    if (listen(sockfd, MAX_CLIENTS) < 0) {
      perror("listen() - 요청 대기 실패");
      close(sockfd);
      exit(1);
    }

    printf("서버가 포트 %d에서 연결 요청을 기다리고 있음...\n", portno);

    close(sockfd);

    return 0;
}
```

### 4. `accept()`, 클라이언트 연결 요청 수락

```c
#include <stdio.h>      // printf, perror
#include <stdlib.h>     // exit
#include <string.h>     // bzero, memset
#include <sys/socket.h> // socket
#include <unistd.h>     // close
#include <sys/types.h>  // accept
#include <netinet/in.h> // bind

#define MAX_CLIENTS 5
#define DEFAULT_PORT 10000

int main(int argc, char *argv[]) {
    /*
    * sockfd : 서버 소켓 디스크립터 (연결 요청을 기다리는 소켓)
    * newsockfd : 새로운 소켓 디스크립터 (연결된 클라이언트와 소통하기 위한 소켓)
    * portno : 포트 번호
    * buffer : 클라이언트 메시지 수신 버퍼
    */
    int sockfd, newsockfd;
    int portno = DEFAULT_PORT;

    char buffer[256];

    /* 
    * sockaddr_in : 서버 주소 정보를 저장하는 구조체
    * cli_addr : 클라이언트 주소 정보를 저장하는 구조체
    */
    struct sockaddr_in serv_addr, cli_addr;

    /*
    * [ socket() 호출 ]
    * AF_INET : 네트워크 주소 체계를 알려준다. 'AF_INET'은 32비트 주소를 사용한다는 뜻이다(IPv4)
    * type : 소켓이 어떤 타입(TCP/UDP 등)인지 알려준다. 'SOCK_STREAM'은 TCP 프로토콜을 사용한 통신 소켓이라는 뜻이다.
    * protocol : 프로토콜 정보; 앞의 두개 인자를 이용해 프로토콜을 특정할 수 없을 때에만 따로 명시해준다. 그 이외에는 예시에서 처럼 0으로 처리해줘도 된다.
    */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    /* socket() 호출 실패 시 오류 처리 */
    if (sockfd < 0) {
        perror("socket() - 소켓 생성 실패");
        exit(1);
    }

    /* 
    * [ 서버 주소 구조체 초기화 ]
    * bzero() 함수로 serv_addr 구조체의 모든 바이트 영역을 0으로 초기화한다. (왜? 안하면 가비지 데이터가 들어갈 수도 있어서)
    * 포트 번호는 명령어의 인자로 받아서 정수로 변환한다. 없을 경우 default 포트 10000번을 사용한다.
    * sin_family: 주소 체계(IPv4)
    * sin_addr.s_addr: 서버 IP 주소(INADDR_ANY: 모든 IP 주소)
    * sin_port: 서버 포트 번호(바이트 오더 변환)
    */
    memset(&serv_addr, 0, sizeof(serv_addr));
    if (argc < 2) {
        printf("포트 번호가 지정되지 않음 - 기본 포트 %d번 사용\n", portno);
    } else {
        portno = atoi(argv[1]);
    }
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(portno);
    
    /* 
    * [ bind() 호출 실패 시 오류 처리 ]
    * bind() 함수는 소켓에 주소를 바인딩하는 함수이다.
    * 소켓 디스크립터, 서버 주소 구조체, 서버 주소 구조체 크기를 인자로 받는다. 
    */
    if (bind(sockfd, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind() - 주소 바인딩 실패");
        close(sockfd);
        exit(1);
    }

    /* 
    * [ listen() 호출 ]
    * listen() 함수는 소켓을 수동 모드로 전환하고, 연결 요청을 대기 상태로 전환한다.
    * 소켓 디스크립터, 연결 요청 대기 큐의 최대 길이를 인자로 받는다.
    */
    if (listen(sockfd, MAX_CLIENTS) < 0) {
      perror("listen() - 요청 대기 실패");
      close(sockfd);
      exit(1);
    }

    printf("서버가 포트 %d에서 연결 요청을 기다리고 있음...\n", portno);

    /* accept() 호출 
    * accept() 함수는 연결 요청을 수락하고, 새로운 소켓 디스크립터를 반환한다.
    * 소켓 디스크립터, 클라이언트 주소 구조체, 클라이언트 주소 구조체 크기를 인자로 받는다.
    * 1. 누군가(sockfd가 listen 중인 포트로) 연결을 요청함
    * 2. accept()가 그 요청을 수락함
    * 3. 운영체제가 자동으로 "새로운 소켓"을 생성해서
    * 4. 그 소켓의 디스크립터를 newsockfd로 리턴해줌
    */
    socklen_t clilen = sizeof(cli_addr); //클라이언트 주소 구조체 크기 
    int newsockfd = accept(sockfd, (struct sockaddr *) &cli_addr, &clilen);
    if (newsockfd < 0) {
        perror("accept() - 연결 수락 실패");
        close(sockfd);
        exit(1);
    }

    printf("클라이언트 연결 수락됨! (newsockfd = %d)\n", newsockfd);

    close(sockfd);

    return 0;
}

```

### 4. `send/recv or read/write`, 메세지 송수신
서버가 클라이언트와 연결을 맺으면, 서로 데이터를 주고받는 송수신 단계(send/recv 또는 write/read) 로 갈수 있다.

클라이언트 ↔ 서버 간 데이터를 주고받으려면, recv() / send() 또는 read() / write() 함수를 사용하면 된다.  
둘 다 기능은 비슷한데, 지금은 TCP 소켓이니까 recv() / send()를 사용한다.  








---

## 의문점 정리

### 1. 파일 디스크립터는 뭘 의미하는 건가?
사전적 정의는 다음과 같다.  
> 파일 디스크립터(File Descriptor, FD)는 "열린 파일(또는 자원)에 대한 커널 내부 핸들(번호)"이다.
이는 권한은 아니고 **참조용 주소표** 같은 것이다.

즉 아래와 같이 이해하면 된다.
> 파일 디스크립터는 운영체제가 열린 파일이나 소켓 등을 구분하기 위해 부여하는 정수 번호.

예를 들어보자.
```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
```
- OS가 내부적으로 소켓을 생성하고 
- 그 소켓을 참조하기 위한 숫자 (예: 3) 를 리턴
- 이후 코드에서는 그 숫자만 가지고 OS에 명령을 내리는 것이다.

```c
bind(sockfd, ...);   // ← 소켓 3번에 포트 바인딩
write(sockfd, ...);  // ← 소켓 3번으로 데이터 전송
```

### 2. 바이트 순서
> 엔디언(Endian)은 프로그래밍에서 데이터를 메모리에 저장하는 방식을 뜻한다.  
> 컴퓨터 시스템에서 데이터를 저장하고 표현하는 방식을 나타내는 개념으로, 바이트 순서를 의미합니다. 
```c
    memset(&serv_addr, 0, sizeof(serv_addr));
    portno = atoi(argv[1]);
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(portno);
```
위 코드에서 `serv_addr.sin_port = htons(portno);`이 부분은 **포트 번호를 네트워크 바이트 순서로 변환**해주는 과정이라고 한다.

왜 이런 과정이 필요하냐면, 
- 컴퓨터가 Little-endian 방식으로 데이터를 저장하더라도, 네트워크에 보낼 땐 모두 Big-endian으로 맞춰야 통신이 제대로 되기 때문.
- CPU, 네트워크 등에서 바이트를 저장하는 순서에 차이가 있을 수 있음.
- 두 호스트가 데이터를 주고받을 때 저장하는 순서의 타입을 맞춰야 제대로 처리할 수 있음.

### 3. newsockfd는 socket()을 통해 생성할 필요가 없나?
newsockfd는 클라이언트와 통신하는 소켓인데, 그럼 socket()을 통해 직접 생성해줘야 하는 거 아닌가?
-> accept() 함수가 자동으로 socket을 만들어 주기 때문에 하지 않아도 된다!

```c
int newsockfd = accept(sockfd, (struct sockaddr *) &cli_addr, &clilen);
```

이 한 줄에서 일어나는 일은 아래와 같다.

1. 누군가(sockfd가 listen 중인 포트로) 연결을 요청함
2. accept()가 그 요청을 수락함
3. 운영체제가 자동으로 "새로운 소켓"을 생성해서
4. 그 소켓의 디스크립터를 newsockfd로 리턴해줌
