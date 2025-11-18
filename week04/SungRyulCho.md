Spring Boot 백엔드 개발 스택(Java, JPA, Spring)의 관점에서 HTTP(Hypertext Transfer Protocol)의 핵심 요소를 설명합니다.

## 📥 1. 개념 정의

**HTTP(Hypertext Transfer Protocol)**는 분산 하이퍼미디어 환경에서 데이터를 교환하기 위한 애플리케이션 계층의 **표준 통신 규약(Protocol)**입니다.

Spring Boot 개발 환경에서 HTTP의 역할은 다음과 같이 정의됩니다.

- **서버 (Server):** 내장된 **톰캣(Tomcat) 서블릿 컨테이너**가 HTTP 서버의 역할을 수행하며, 클라이언트의 연결 요청(Connection)을 수신하고 처리합니다.
- **클라이언트 (Client):** 웹 브라우저, 모바일 애플리케이션(iOS/Android) 또는 MSA(Microservice Architecture) 환경의 다른 서비스가 될 수 있습니다.
- **전송 데이터 (Data):** 과거 HTML이 중심이었으나, 현대의 RESTful API 환경에서는 **JSON(JavaScript Object Notation)** 형식이 데이터 교환의 표준으로 사용됩니다.

## 🧐 2. 필요성

비즈니스 로직(Service Layer)과 데이터 영속성(Persistence Layer, JPA/Repository)이 아무리 견고하게 구현되어도, 외부 클라이언트가 이를 호출하고 상호작용할 표준화된 **진입점(Entry Point)**이 없다면 애플리케이션은 고립됩니다.

HTTP는 이 진입점의 '언어'를 정의합니다. Spring Boot에서는 **'디스패처 서블릿(DispatcherServlet)'**이 **프론트 컨트롤러(Front Controller)** 패턴으로 모든 HTTP 요청을 단일 지점에서 수신한 뒤, `HandlerMapping`을 통해 가장 적절한 **`@Controller`**의 메서드로 요청을 라우팅합니다.

---

## 🛠️ 3. 핵심 사용법 (Spring Boot Code Examples)

Spring 애플리케이션에서 HTTP 요청을 직접 처리하는 컴포넌트는 **`@RestController`** 어노테이션이 적용된 클래스입니다.

### ➡️ 1. `GET` 요청 처리 (데이터 조회)

클라이언트가 특정 리소스의 조회를 요청합니다. (예: `GET /api/users/1`)

- **`@GetMapping`**: HTTP `GET` 메서드 요청을 특정 핸들러 메서드에 매핑합니다.
- **`@PathVariable`**: URL 경로 세그먼트(Path Segment)에 포함된 변수(예: `1`)를 메서드 파라미터(예: `Long id`)에 바인딩합니다.
- **`ResponseEntity`**: HTTP 응답의 상태 코드, 헤더, 그리고 본문(Body)을 세밀하게 제어할 수 있는 Spring 제공 객체입니다.

```java
// /src/main/java/com/example/demo/controller/UserController.java

@RestController
@RequestMapping("/api/users") // 이 컨트롤러의 모든 핸들러는 "/api/users" 네임스페이스 하위에 위치
@RequiredArgsConstructor // Lombok: final 필드에 대한 생성자 주입
public class UserController {

    private final UserService userService; // 비즈니스 로직은 Service 레이어로 위임

    /*
     * 특정 ID의 사용자 정보를 조회합니다. (Read)
     * HTTP 200 OK 또는 404 Not Found를 반환합니다.
     */

    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDto> getUserById(@PathVariable Long id) {
        try {
            // Service -> Repository(JPA)를 통해 엔티티 조회
            User user = userService.findById(id);

            // Entity를 Response DTO로 변환 (중요: Entity를 직접 노출하지 않음)
            UserResponseDto responseDto = new UserResponseDto(user);

            // HTTP 200 OK 상태 코드와 DTO 본문(JSON) 반환
            return ResponseEntity.ok(responseDto);

        } catch (EntityNotFoundException e) {
            // JpaRepository의 findById().orElseThrow() 등에서 발생
            // 리소스가 존재하지 않으므로 HTTP 404 Not Found 반환
            return ResponseEntity.notFound().build();
        }
    }
}
```

### 2. `POST` 요청 처리 (리소스 생성)

클라이언트가 HTTP Body에 데이터를 담아 새로운 리소스 생성을 요청합니다. (예: `POST /api/users`)

- **`@PostMapping`**: `POST` 메서드 요청을 매핑합니다.
- **`@RequestBody`**: HTTP Request Body에 포함된 JSON 데이터를 지정된 Java 객체(DTO)로 **역직렬화(Deserialization)**합니다. (Spring의 `Jackson` 라이\_브러리가 기본으로 수행)
- **`@Valid`**: (Optional) `spring-boot-starter-validation` 의존성이 있을 경우, DTO 내부에 정의된 제약 조건(예: `@NotBlank`, `@Email`)을 검증합니다.

```java
// /src/main/java/com/example/demo/controller/UserController.java

    /**
     * 신규 사용자를 생성합니다. (Create)
     * HTTP 201 Created를 반환합니다.
     */
    @PostMapping
    public ResponseEntity<UserResponseDto> createUser(@RequestBody @Valid UserCreateRequestDto requestDto) {

        // 1. @RequestBody를 통해 JSON -> DTO로 자동 변환 및 검증 완료

        // 2. DTO를 Entity로 변환하여 Service 레이어에 생성 요청
        User newUser = userService.createUser(requestDto.toEntity());

        // 3. 생성된 Entity를 Response DTO로 변환
        UserResponseDto responseDto = new UserResponseDto(newUser);

        // 4. REST 원칙에 따라, 생성된 리소스의 URI를 Location 헤더에 담아
        //    HTTP 201 Created 상태 코드로 응답
        URI location = URI.create("/api/users/" + newUser.getId());
        return ResponseEntity.created(location).body(responseDto);
    }
```

### 🔄 3. `PUT` / `DELETE` 요청 처리 (수정 및 삭제)

- **`@PutMapping("/{id}")`**: 리소스 **전체 수정(Update)**. (부분 수정은 `PATCH` 사용)
- **`@DeleteMapping("/{id}")`**: 리소스 **삭제(Delete)**.
- **`ResponseEntity.noContent().build()`**: `DELETE` 성공 시, 클라이언트에게 반환할 본문이 없음을 의미하는 **HTTP 204 No Content** 상태 코드를 반환하는 것이 표준적인 방법입니다.

```java
// /src/main/java/com/example/demo/controller/UserController.java (계속)

    /**
     * 기존 사용자 정보를 수정합니다. (Update)
     * HTTP 200 OK를 반환합니다.
     */
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDto> updateUser(
            @PathVariable Long id,
            @RequestBody @Valid UserUpdateRequestDto requestDto) {

        User updatedUser = userService.updateUser(id, requestDto);
        return ResponseEntity.ok(new UserResponseDto(updatedUser));
    }

    /**
     * 기존 사용자를 삭제합니다. (Delete)
     * HTTP 204 No Content를 반환합니다.
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);

        // 성공적으로 삭제되었으며, 반환할 본문이 없음을 명시
        return ResponseEntity.noContent().build();
    }
```

## 🔬 4. 심화 학습 (Spring 관점)

### 🔑 인증 (Authentication)과 HTTP

HTTP는 본질적으로 **무상태성(Stateless)** 프로토콜입니다. 즉, 각 요청은 독립적입니다. 그러나 '로그인 상태 유지'는 상태가 필요합니다. Spring Security는 이를 두 가지 방식으로 주로 해결합니다.

1.  **세션/쿠키 방식 (Stateful):**

    - **로그인 (`POST /login`):** `Spring Security`가 인증 성공 시, 서버 메모리에 `HttpSession`을 생성하고, 이 세션에 접근할 수 있는 `JSESSIONID`를 **`Set-Cookie`**라는 **HTTP 응답 헤더**에 담아 클라이언트에 전송합니다.
    - **이후 요청 (`GET /api/mypage`):** 브라우저는 모든 후속 요청에 `JSESSIONID`를 **`Cookie`**라는 **HTTP 요청 헤더**에 담아 자동으로 전송합니다.
    - **서버:** `Cookie` 헤더의 `JSESSIONID`를 확인하여 `HttpSession`을 조회, 사용자를 식별합니다.

2.  **토큰 기반 방식 (Stateless - JWT):**
    - **로그인 (`POST /login`):** 서버가 인증 성공 시, 사용자의 정보를 담은 암호화된 토큰(JWT)을 생성하여 **HTTP 응답 Body(JSON)**로 전송합니다.
    - **이후 요청 (`GET /api/mypage`):** 클라이언트(브라우저/앱)는 발급받은 토큰을 **`Authorization: Bearer <token...>`** 형태의 **HTTP 요청 헤더**에 직접 담아 전송합니다.
    - **서버:** `Authorization` 헤더를 파싱하고 토큰의 유효성을 검증하여 사용자를 식별합니다.

### 🌐 RESTful API 설계

위 3번 섹션의 예시는 HTTP를 'RESTful'하게 사용하려는 시도입니다. **REST(Representational State Transfer)**는 HTTP의 잠재력을 최대한 활용하기 위한 아키텍처 스타일입니다.

- **리소스(명사) 중심 URL:** `/users`, `/users/1` (O) / `/getUser`, `/createUser` (X)
- **행위(동사)는 HTTP 메서드로 표현:** `GET` (조회), `POST` (생성), `PUT` (수정), `DELETE` (삭제)

## ⚠️ 5. 실전 주의사항 (Spring Backend 관점)

### 1. [Security] 민감 정보의 전송 방식

`GET` 요청의 `@PathVariable`이나 `@RequestParam`을 통해 민감 정보(비밀번호, 개인 식별 정보 등)를 전송하는 것은 심각한 보안 취약점입니다.

```java
// [안티 패턴: Anti-Pattern]
// GET /api/login?username=my_id&password=my_secret_password123
@GetMapping("/login")
public String login(@RequestParam String username, @RequestParam String password) {
    // 🚨 URL에 민감 정보가 노출되며,
    // 🚨 웹 서버 접근 로그(Access Log)에 평문으로 기록됩니다.
}
```

**[Best Practice]** 모든 민감 정보는 `POST` 또는 `PUT`의 **HTTP Body**에 담아 **HTTPS** 프로토콜을 통해 암호화 전송해야 합니다.

### 2. [CORS] 동일 출처 정책 (Same-Origin Policy)

웹 브라우저는 보안상의 이유로 **SOP(Same-Origin Policy)**를 적용합니다. 이로 인해 `http://localhost:3000` (프론트엔드)에서 `http://localhost:8080` (Spring Boot API)으로의 비동기 요청(AJAX, `fetch`)은 기본적으로 차단됩니다.

**[해결책 (Spring Boot)]** 서버가 "특정 도메인(Origin)의 요청을 허용한다"는 **HTTP 응답 헤더**를 전송해야 합니다. (`@CrossOrigin` 어노테이션 또는 `WebMvcConfigurer`를 통한 전역 설정)

```java
// /src/main/java/com/example/demo/config/WebConfig.java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**") // /api/ 경로 하위에만 CORS 정책 적용
            .allowedOrigins("http://localhost:3000") // 🚨 허용할 프론트엔드 Origin
            .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS")
            .allowCredentials(true) // 쿠키를 포함한 요청 허용
            .maxAge(3600); // Pre-flight 요청의 캐시 시간 (초)
    }
}
```

### 3. [Exception] 전역 예외 처리 (Global Exception Handling)

Service 레이어에서 발생한 `EntityNotFoundException`이나 `@Valid` 검증 실패(`MethodArgumentNotValidException`)를 컨트롤러가 처리하지 않으면, **HTTP 500 Internal Server Error**와 함께 처리되지 않은 스택 트레이스가 클라이언트에 노출될 수 있습니다.

**[Best Practice]** `@RestControllerAdvice`와 `@ExceptionHandler`를 사용하여 예외를 전역적으로 처리하고, 명시적인 HTTP 상태 코드와 일관된 에러 포맷(JSON)을 반환해야 합니다.

```java
// /src/main/java/com/example/demo/exception/GlobalExceptionHandler.java

@RestControllerAdvice // 모든 @RestController의 예외를 중앙에서 처리
public class GlobalExceptionHandler {

    // JPA 리소스 조회 실패 시
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEntityNotFound(EntityNotFoundException ex) {
        ErrorResponse response = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND); // HTTP 404
    }

    // @Valid 검증 실패 시
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        String errorMessage = ex.getBindingResult().getAllErrors().get(0).getDefaultMessage();
        ErrorResponse response = new ErrorResponse("BAD_REQUEST", errorMessage);
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST); // HTTP 400
    }

    // (ErrorResponse는 별도로 정의한 에러 응답 DTO 클래스)
}
```
