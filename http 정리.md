# http

## HTTP란?

- **HTTP(HyperText Transfer Protocol)** : 웹에서 **클라이언트(브라우저, 앱)** 와 **서버**가 데이터를 주고받는 통신 규칙(약속)
- **브라우저가 서버에게 말하는 언어**
- 웹의 모든 통신은 HTTP로 이루어진다.

---

## HTTP의 기본 구조

```
클라이언트(브라우저)
     ↓ 요청(Request)
서버(백엔드)
     ↑ 응답(Response)

```

- 클라이언트가 요청을 보내면 서버가 응답을 보낸다.
- 요청과 응답이 끝나면 연결이 끊어진다.
- 이 규칙을 통해 페이지를 보거나, 로그인, 회원가입 등을 할 수 있다.

---

## 요청(Request)

요청은 **클라이언트 → 서버로 보내는 메시지**

| 구성 요소 | 설명 | 예시 |
| --- | --- | --- |
| **요청 라인** | 요청의 종류와 목적 | `GET /index.html HTTP/1.1` |
| **헤더(Header)** | 부가 정보 (브라우저 정보, 데이터 형식 등) | `User-Agent: Chrome`, `Content-Type: application/json` |
| **바디(Body)** | 실제 보낼 데이터 (POST 시 주로 사용) | `id=taehun&pw=1234` |

---

## 응답(Response)

응답은 **서버 → 클라이언트로 보내는 메시지**

| 구성 요소 | 설명 | 예시 |
| --- | --- | --- |
| **상태 라인** | 처리 결과 | `HTTP/1.1 200 OK` |
| **헤더(Header)** | 응답 관련 정보 | `Content-Type: text/html; charset=UTF-8` |
| **바디(Body)** | 실제 데이터 (HTML, JSON 등) | `<html>...</html>` |

---

## HTTP 메서드

| 메서드 | 설명 | 예시 |
| --- | --- | --- |
| **GET** | 서버의 데이터를 가져온다 | 게시글 목록 보기 |
| **POST** | 서버에 데이터를 보낸다 | 회원가입, 로그인 |
| **PUT** | 기존 데이터를 전체 수정한다 | 프로필 전체 변경 |
| **PATCH** | 일부만 수정한다 | 닉네임만 변경 |
| **DELETE** | 데이터를 삭제한다 | 게시글 삭제 |

---

## 상태 코드 (Status Code)

| 코드 | 의미 | 설명 |
| --- | --- | --- |
| **200 OK** | 성공 | 요청이 정상 처리됨 |
| **201 Created** | 생성 성공 | 새 데이터가 만들어짐 |
| **400 Bad Request** | 잘못된 요청 | 요청 형식이나 데이터 오류 |
| **401 Unauthorized** | 인증 필요 | 로그인되지 않음 |
| **403 Forbidden** | 접근 금지 | 권한 없음 |
| **404 Not Found** | 찾을 수 없음 | 잘못된 주소 |
| **500 Internal Server Error** | 서버 오류 | 코드 문제 발생 |

---

## 주요 헤더(Header)

| 헤더 | 역할 | 예시 |
| --- | --- | --- |
| **Host** | 요청 대상 서버 | `Host: example.com` |
| **User-Agent** | 클라이언트 정보 | `Chrome/129` |
| **Content-Type** | 데이터 형식 | `application/json`, `text/html` |
| **Accept** | 클라이언트가 원하는 응답 형식 | `application/json` |
| **Authorization** | 인증 정보 (JWT 등) | `Bearer eyJhbGci...` |
| **Set-Cookie** | 서버가 쿠키 저장 요청 | `Set-Cookie: sessionId=abc123; HttpOnly` |
| **Cookie** | 클라이언트가 쿠키 전달 | `Cookie: sessionId=abc123` |

---

## HTTP의 특징

| 특징 | 설명 |
| --- | --- |
| **비연결성 (Connectionless)** | 요청–응답이 끝나면 연결이 끊어진다. |
| **무상태성 (Stateless)** | 서버는 이전 요청 정보를 기억하지 않는다. |
| **단순함** | 메서드와 상태코드만으로 통신 가능하다. |
| **확장성** | 새 헤더나 메서드를 쉽게 추가할 수 있다. |

💡 서버가 사용자의 상태를 유지하려면 → **쿠키(Cookie)** 나 **세션(Session)** 을 사용한다.

---

## HTTP vs HTTPS

| 구분 | HTTP | HTTPS |
| --- | --- | --- |
| 암호화 | ❌ 없음 | ✅ 있음 (TLS/SSL 사용) |
| 포트번호 | 80 | 443 |
| 보안 수준 | 낮음 | 높음 |
| 사용 예시 | 테스트, 로컬 개발 | 로그인, 결제 등 보안 필요한 서비스 |

---

## 예시 코드 (Spring에서 HTTP 다루기)

```java
@PostMapping("/login")
public void login(
    @RequestBody String body,                   // 요청 바디 읽기
    @RequestHeader("content-type") String type, // 요청 헤더 읽기
    @CookieValue(value = "JSESSIONID", required = false) String cookie // 쿠키 읽기
) {
    System.out.println(type);
    System.out.println(cookie);
    System.out.println(body);
}

```

---

## 정리

- HTTP는 **클라이언트와 서버가 대화하는 약속된 규칙**
- 요청(Request) → “이거 해줘요”
- 응답(Response) → “이게 결과야”
- 메서드로 동작 구분 (GET, POST, PUT, DELETE 등)
- 상태코드로 결과 구분 (200, 404, 500 등)
- HTTPS는 HTTP에 **보안을 추가한 버전**