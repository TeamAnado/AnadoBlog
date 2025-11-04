# HTTP 통신: 원리에서 최적화까지 — 엔지니어링 관점으로 보는 웹의 언어

## 1. HTTP의 구조와 동작 원리 — “웹의 언어를 해석하기”

HTTP(HyperText Transfer Protocol)는 웹의 기반이 되는 통신 규약이다. 하지만 많은 개발자들이 “요청을 보낸다”, “응답을 받는다” 정도로만 이해하고 있다. 엔지니어링 관점에서 중요한 건 **HTTP가 단순한 텍스트 교환 이상의 것**이라는 점이다.

HTTP의 본질은 **요청(Request)과 응답(Response)** 의 조합이다. 이 두 메시지는 서로 명확한 규약에 따라 주고받으며, 각각 다음과 같은 구성 요소를 가진다.

### Request 구조
```
GET /users HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer <token>
```

- **Request Line**: 어떤 리소스에 어떤 메서드로 접근할지 정의 (GET /users)  
- **Headers**: 통신에 필요한 부가 정보 (Accept, Authorization 등)  
- **Body**: 데이터가 필요한 요청(POST, PUT)일 경우 전송되는 내용  

### Response 구조
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 45

{"id":1,"name":"Ricky","role":"developer"}
```

- **Status Line**: 서버가 요청을 어떻게 처리했는지를 나타냄  
- **Headers**: 응답 정보(타입, 캐싱 정책 등)  
- **Body**: 실제 응답 데이터(JSON 등)  

HTTP는 기본적으로 **Stateless(무상태)** 하다. 즉, 요청 하나가 끝나면 서버는 그 클라이언트를 “기억하지 않는다.” 이 단순함 덕분에 확장성은 뛰어나지만, 반대로 **인증이나 세션 관리의 복잡도**가 생긴다. 이를 해결하기 위해 Cookie, Session, Token 같은 개념이 등장하게 된다.

> HTTP의 기본 구조를 제대로 이해해야 REST, GraphQL, gRPC 등 상위 레이어의 통신 방식을 온전히 이해할 수 있다.


## 2. 요청/응답 설계 — “API 통신의 실제”

HTTP를 가장 많이 마주치는 순간은 REST API 설계다. 이때 단순히 “경로를 만든다” 수준을 넘어, **“리소스 중심으로 일관된 인터페이스를 설계하는 일”** 을 해야 한다.

예를 들어, 다음 두 API 중 어느 쪽이 더 좋은 설계일까?

| 나쁜 예 | 좋은 예 |
|----------|----------|
| `/getUserInfo?id=1` | `/users/1` |
| `/createUser` | `POST /users` |
| `/updateUserName` | `PATCH /users/1/name` |

HTTP는 메서드 자체로 이미 “의도”를 표현한다.  
- `GET`: 리소스 조회  
- `POST`: 리소스 생성  
- `PUT` / `PATCH`: 리소스 수정  
- `DELETE`: 리소스 삭제  

이 규칙만 잘 지켜도 URI에 동사(get, update 등)를 넣을 필요가 없다. 즉, URI는 **“무엇(what)”**, 메서드는 **“행위(action)”** 를 표현한다.

또한, 요청과 응답을 설계할 때 가장 많이 놓치는 부분 중 하나가 **헤더(Header)** 다. 헤더는 단순한 부가정보가 아니라 **통신 프로토콜 계약서**다.

```http
POST /login HTTP/1.1
Content-Type: application/json
Accept: application/json
```

서버와 클라이언트가 어떤 형식으로 데이터를 주고받을지 합의하는 부분이며, 여기서 조금만 어긋나도 클라이언트는 응답을 파싱하지 못한다. 즉, HTTP 헤더를 제대로 이해하는 것이 곧 **API 신뢰성 확보의 출발점**이다.


## 3. HTTP 통신의 성능과 신뢰성 — “느리지 않고 안전한 웹 만들기”

HTTP의 구조를 이해하고 설계까지 잘 했더라도, 실제 서비스에서는 또 다른 문제가 등장한다.  
“왜 이렇게 느리지?”  
“왜 가끔 타임아웃이 나지?”  
이건 대부분 **네트워크 레벨의 최적화 문제**다.

### 1) Keep-Alive와 연결 재사용
HTTP 1.0 시절엔 요청 하나마다 TCP 연결을 새로 맺었다. 이건 RTT(round trip time)가 큰 환경에서 치명적이다.  
1.1부터는 `Connection: keep-alive` 덕분에 **연결 재사용**이 가능해졌고, HTTP/2에서는 **Multiplexing**으로 여러 요청을 하나의 연결에서 동시에 보낼 수 있다.

### 2) HTTPS와 TLS Handshake
HTTPS는 단순히 ‘보안 버전의 HTTP’가 아니다. TLS Handshake 과정에서 암호화 키를 교환하고, 인증서로 서버의 신뢰성을 검증한다. 이 과정은 3~4번의 왕복(RTT)을 요구하므로, 성능 면에서는 느리지만 **보안과 신뢰성을 대가로 얻는 절충**이다.

```text
Client                                 Server
  |                                       |
  | ----------- ClientHello ------------> |
  |                                       |
  | <----------- ServerHello ------------ |
  | <----------- Certificate ------------ |
  | <-------- ServerHelloDone ----------- |
  | --------- ClientKeyExchange---------> |
  | --------- ChangeCipherSpec ---------> |
  | --------- EncryptedHandshake -------> |
  | <-------- ChangeCipherSpec ---------- |
  | <-------- EncryptedHandshake -------- |
  |                                       |
         (Secure Channel Established)     
```
### 3) 캐시 전략과 CDN
정적 자원을 캐싱하면 RTT 자체를 줄일 수 있다.  

- `Cache-Control: max-age=86400`  
- `ETag`와 `If-None-Match`를 이용한 조건부 요청  

또한 CDN(Content Delivery Network)을 활용하면 사용자의 지리적 거리까지 단축된다.

> HTTP의 성능 문제는 코드의 문제가 아니라 **물리적 거리와 프로토콜 효율의 문제**다. 서버 튜닝보다 먼저, **프로토콜 수준에서의 최적화**를 고민해야 한다.