---
title: socket-programing
---

## 소켓 프로그래밍, 대체 뭘까?

안녕하세요 여러분! 네트워크 프로그래밍에 관심이 있거나, 서버-클라이언트 모델을 구현하는 데에 관심이 있는 분들이라면 '소켓(socket)'이라는 단어를 한 번쯤 들어보셨겠죠? 능동적으로 네트워크 관련 코드를 작성해보리라는 야심찬 시작이라면 오늘 블로그 포스트가 큰 도움이 될 거예요! 오늘은 특히 **서버 소켓**을 만드는 과정을 중점적으로 다루려고 합니다. 자, 그러면 소켓 프로그래밍의 세계로 깊이 들어가 보죠!

### 개념 짚고 가기

먼저 '소켓'에 관한 기본적인 개념을 이해하고 지나가야 합니다. 소켓은 네트워크 양쪽 끝단에서 동작하는 소프트웨어 장치로, 주로 서버와 클라이언트 사이의 통신을 위해 사용됩니다. 소켓을 통해 네트워크 상의 다른 컴퓨터와 데이터를 주고받을 수 있습니다.

#### 관련된 함수들 및 구조체
- **socket()**: 소켓을 생성하는 함수입니다.
- **bind()**: 생성한 소켓을 특정 IP와 포트에 묶어줍니다.
- **listen()**: 해당 IP와 포트로 들어오는 연결 요청을 대기 상태로 만듭니다.
- **accept()**: 클라이언트의 연결 요청을 받아들이고, 연결된 소켓을 새로 생성합니다.
- **setSockOpt()**: 소켓 옵션을 설정하는 함수입니다.
- **setNonBlocking()**: 소켓을 논블로킹 모드로 설정합니다.

### 소켓 관련함수
#### 1. socket()
```cpp
int socket(int domain, int type, int protocol);
- domain: 통신 영역 (예: AF_INET for IPv4, AF_INET6 for IPv6)
- type: 통신 타입 (예: SOCK_STREAM for TCP, SOCK_DGRAM for UDP)
- protocol: 프로토콜 (대부분 0으로 설정)
```
|프로토콜체계(Protocol Family)|  정의  |
|------|------|
|PF_INET|IPv4 인터넷 프로토콜|
|PF_INET6|IPv6 인터넷 프로토콜|
|PF_LOCAL|Local 통신을 위한 UNIX 프로토콜|
|PF_PACKET|Low level socket을 위한 인터페이스|

**AF_ vs PF**
궁금하신분은 아래의 링크를 참조해주세요.
[AF_INET vs PF_INET](https://www.bangseongbeom.com/af-inet-vs-pf-inet.html)
**리눅스**에서는 차이가 없다고 생각하시면 됩니다.

#### 2. bind()
```cpp
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
- sockfd: 소켓 파일 디스크립터
- addr: 바인딩할 주소 정보
- addrlen: 주소 구조체의 길이
```
#### 3. listen()
```cpp
int listen(int sockfd, int backlog);
- sockfd: 소켓 파일 디스크립터
- backlog: 연결 대기열의 최대 길이
```
#### 4. accept()
```cpp
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
- sockfd: 리스닝 소켓 파일 디스크립터
- addr: 클라이언트 주소 정보를 저장할 구조체 or NULL
- addrlen: 주소 구조체의 길이 or NULL
```

#### 5. setsockopt()
```cpp
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
- sockfd: 소켓 파일 디스크립터
- level: 옵션을 설정할 프로토콜 레벨
- optname: 설정할 옵션의 이름
- optval: 옵션 값 -> true or false
- optlen: 옵션 값의 길이
```

```cpp
int opt = 1; -> true
setsockopt(socket, SOL_SOCKET, (const char*)&opt, sizeof(opt));
```
|값|자료형|설명|
|-----|----|----|
|<span style="color:#b1ffff"> SO_REUSEADDR</span>|	BOOL	|이미 사용된 주소를 재사용하도록 함|
|<span style="color:#b1ffff">SO_RCV_BUF</span>|	int		|수신용 버퍼의 크기 지정
|<span style="color:#b1ffff">SO_SND_BUF</span>|	int		|송신용 버퍼의 크기 지정
|<span style="color:#b1ffff">SO_RECVTIMEO</span>|	DWORD(timeval)	|수신시 Blocking 제한 시간을 설정
|<span style="color:#b1ffff">SO_SNDTIMEO</span>|	DWORD(timeval)	|송신시 Blocking 제한 시간을 설정
|<span style="color:#b1ffff">SO_KEEPALIVE</span>|	BOOL	|TCP 통신에서만 유효. 일정 시간마다 연결 유지 상태를 체크.
|<span style="color:#b1ffff">SO_LINGER</span>	|struct LINGER	|소켓을 닫을 때 남은 데이터의 처리 규칙 지정
|<span style="color:#b1ffff">SO_DONTLINGER</span>|	BOOL	|소켓을 닫을때 남은 데이터를 보내기 위해서 블럭되지 않도록 함
|<span style="color:#b1ffff">SO_DONTROUTE</span>|	BOOL	|라우팅(Routing)하지 않고 직접 인터페이스로 전송
|<span style="color:#b1ffff">SO_BROADCAST</span>|	BOOL	|브로드캐스트 사용 가능 여부

### struct addrinfo vs struct sockaddr_in

#### struct addrinfo
```cpp
struct addrinfo {
    int              ai_flags;
    int              ai_family;
    int              ai_socktype;
    int              ai_protocol;
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;
    char            *ai_canonname;
    struct addrinfo *ai_next;
};
```
addrinfo는 getaddrinfo() 함수와 함께 사용되며, 호스트 이름과 서비스 이름을 소켓 주소 구조체로 변환하는 데 유용합니다. IPv4와 IPv6를 모두 지원합니다.

#### struct sockaddr_in
```cpp
struct sockaddr_in {
    sa_family_t    sin_family;
    in_port_t      sin_port;
    struct in_addr sin_addr;
};
```
sockaddr_in은 IPv4 전용 구조체로, 더 간단하지만 IPv6를 지원하지 않습니다.

### 주요 차이점
1. addrinfo는 프로토콜에 독립적이며, IPv4와 IPv6를 모두 지원합니다.
2. sockaddr_in은 IPv4에 특화되어 있어 사용이 간단합니다.
3. addrinfo는 링크드 리스트 형태로 여러 주소 정보를 저장할 수 있습니다.

---



이제 코드 예제와 함께 살펴보겠습니다. 여러분의 이해를 돕기 위해 간단한 서버 소켓 클래스를 예제로 들어볼게요.

### 서버 소켓 클래스 만들기

#### 소켓 생성
가장 먼저 할 일은 소켓을 생성하는 일입니다. 

```cpp
void createSocket() {
    _serverSocket = ::socket(AF_INET, SOCK_STREAM, 0);
    if (_serverSocket == -1) {
        logError("Failed to create socket");
        throw SocketException("Failed to create socket");
    }
}
```

`socket()` 함수는 네트워크 소켓을 생성하는 역할을 합니다. `AF_INET`은 IPv4를 의미하고, `SOCK_STREAM`은 TCP 프로토콜을 사용한다는 뜻입니다. 

#### 소켓 바인딩
이제 `bind()` 함수를 통해 소켓을 특정 IP와 포트에 묶어봅시다.

```cpp
void bindSocket() {
    struct addrinfo hints, *serverInfo;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_INET;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_protocol = IPPROTO_TCP;

    int status = getaddrinfo(_host, _port, &hints, &serverInfo);
    if (status != 0) {
        logError("getaddrinfo failed");
        throw SocketException("getaddrinfo failed");
    }

    if (::bind(_serverSocket, serverInfo->ai_addr, serverInfo->ai_addrlen) == -1) {
        logError("bind failed");
        freeaddrinfo(serverInfo);
        throw SocketException("bind failed");
    }
    freeaddrinfo(serverInfo);
}
```

`bind()` 함수는 소켓을 특정 IP와 포트에 연결합니다. `getaddrinfo()` 함수를 통해 주소 정보를 얻은 후 이를 이용해 소켓을 바인딩합니다.

#### 소켓 리스닝
할 일은 이제 들어오는 연결 요청을 대기 상태로 만드는 일입니다.

```cpp
void startListening(int backlog) {
    if (::listen(_serverSocket, backlog) == -1) {
        logError("listen failed");
        throw SocketException("listen failed");
    }
}
```

`listen()` 함수는 소켓을 듣는 상태로 만듭니다. `backlog`는 대기열의 크기를 의미합니다. 

#### 연결 요청 수락
클라이언트의 연결 요청을 받아들이는 함수가 `accept()` 입니다.

```cpp
int acceptConnection() {
    int clientSocket = ::accept(_serverSocket, NULL, NULL);
    if (clientSocket == -1) {
        logError("Failed to accept socket");
        throw SocketException("Failed to accept socket");
    }
    return clientSocket;
}
```

`accept()` 함수는 클라이언트의 연결 요청을 수락하고, 새로 생성한 소켓의 파일 식별자를 반환합니다.

#### 소켓 옵션 설정
오토매틱하게 설정할 수 있는 소켓 옵션들이 존재합니다. 

```cpp
void setSocketOptions() {
    int opt = 1;
    setSockOpt(SOL_SOCKET, SO_REUSEADDR, opt);
    setSockOpt(SOL_SOCKET, SO_KEEPALIVE, opt);
    setSockOpt(SOL_SOCKET, SO_RCVBUF, DEFAULTBUFFER);
}
```

#### 코드 전체 살펴보기

이제까지 설명한 부분들을 종합해서 서버 소켓 클래스를 초기화하고 설정하는 과정을 어떻게 구성할 수 있는지 살펴보겠습니다.

```cpp
class ServerSocket {
public:
    ServerSocket(const std::string &host, const std::string &port)
        : _host(host.c_str()), _port(port.c_str()), _serverSocket(-1) {}

    void initServerSocket() {
        createSocket();
        setNonBlocking(_serverSocket);
        setSocketOptions();
        bindSocket();
        startListening();
    }

    int acceptConnection() {
        struct sockaddr_in clientAddr;
        socklen_t clientLen = sizeof(clientAddr);
        int clientSocket = ::accept(_serverSocket, (struct sockaddr*)&clientAddr, &clientLen);
        if (clientSocket == -1) {
            if (errno != EWOULDBLOCK && errno != EAGAIN) {
                throw SocketException("Failed to accept socket");
            }
        }
        return clientSocket;
    }

private:
    void createSocket() {
        _serverSocket = ::socket(AF_INET, SOCK_STREAM, 0);
        if (_serverSocket == -1) {
            throw SocketException("Failed to create socket");
        }
    }

    void setNonBlocking(int sock) {
        int flags = fcntl(sock, F_GETFL, 0);
        if (flags == -1) {
            throw SocketException("Failed to get socket flags");
        }
        if (fcntl(sock, F_SETFL, flags | O_NONBLOCK) == -1) {
            throw SocketException("Failed to set non-blocking socket");
        }
    }

    void setSocketOptions() {
        int opt = 1;
        if (setsockopt(_serverSocket, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) == -1) {
            throw SocketException("Failed to set SO_REUSEADDR option");
        }
        if (setsockopt(_serverSocket, SOL_SOCKET, SO_KEEPALIVE, &opt, sizeof(opt)) == -1) {
            throw SocketException("Failed to set SO_KEEPALIVE option");
        }
    }

    void bindSocket() {
        struct addrinfo hints, *serverInfo;
        memset(&hints, 0, sizeof(hints));
        hints.ai_family = AF_INET;
        hints.ai_socktype = SOCK_STREAM;
        hints.ai_flags = AI_PASSIVE;

        int status = getaddrinfo(_host, _port, &hints, &serverInfo);
        if (status != 0) {
            throw SocketException("getaddrinfo failed");
        }

        if (::bind(_serverSocket, serverInfo->ai_addr, serverInfo->ai_addrlen) == -1) {
            freeaddrinfo(serverInfo);
            throw SocketException("bind failed");
        }
        freeaddrinfo(serverInfo);
    }

    void startListening() {
        if (::listen(_serverSocket, SOMAXCONN) == -1) {
            throw SocketException("listen failed");
        }
    }

    const char *_host;
    const char *_port;
    int _serverSocket;
};
```

변수명이나 함수명 등을 일반적인 것으로 변경해 코드 전체를 정리했습니다. 이 클래스는 서버 소켓을 초기화하고 클라이언트의 연결을 수락할 수 있도록 구성되었습니다.

### 마무리

오늘은 소켓 프로그래밍의 기본 개념과 함께 서버 소켓 클래스를 만드는 과정을 살펴보았습니다. 소켓 프로그래밍은 네트워크 애플리케이션을 개발하는 데 있어 매우 중요한 기술입니다. 다음에도 더 흥미로운 주제로 찾아뵙겠습니다. 그럼 이상 소켓 프로그래밍의 기초에 대한 블로그 포스트를 마치겠습니다. 감사합니다!

 > 🔥 해당 포스팅은 [Dev.POST](https://devpost-download.framer.website/) 도움을 받아 작성되었습니다.