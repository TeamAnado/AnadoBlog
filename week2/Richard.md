# JVM의 가상 스레드: 동시성 프로그래밍

## 들어가며

수백, 수천 명의 사용자가 동시에 요청을 보내는 상황에서 어떻게 하면 모든 요청을 효율적으로 처리할 수 있을까? 전통적으로 Java에서는 각 요청마다 새로운 스레드를 생성하거나 스레드 풀을 사용해왔다. 하지만 Java 21에서 도입된 **가상 스레드(Virtual Thread)** 는 이런 고민을 완전히 새로운 방식으로 해결한다.

## 스레드란 무엇인가?

스레드는 프로그램이 실행되는 가장 작은 단위로, 컴퓨터는 여러 개의 스레드를 통해 동시에 여러 작업을 수행할 수 있다.

### 전통적인 플랫폼 스레드의 문제점

Java에서 지금까지 사용해온 스레드는 **플랫폼 스레드(Platform Thread)** 라고 부른다. 이는 운영체제의 네이티브 스레드와 1:1로 매핑되는 구조다.

```java path=null start=null
// 전통적인 스레드 생성 방법
Thread thread = new Thread(() -> {
    System.out.println("Hello from platform thread!");
});
thread.start();
```

플랫폼 스레드의 제약사항:

1. **높은 메모리 사용량**: 각 스레드마다 약 2MB의 스택 메모리가 필요함
2. **생성/소멸 비용**: 스레드 생성과 컨텍스트 스위칭에 상당한 오버헤드 발생
3. **제한된 스케일링**: 수천 개 이상의 스레드 생성 시 성능 저하

### 기존 해결책들의 한계

이런 문제들을 해결하기 위해 시도된 기법들:

- **스레드 풀(Thread Pool)**: 미리 생성된 스레드를 재사용
- **비동기 프로그래밍**: CompletableFuture, Reactive Streams 등
- **논블로킹 I/O**: NIO를 활용한 이벤트 기반 처리

하지만 이런 방법들은 코드의 복잡성을 크게 증가시키는 문제가 있었다.

## 가상 스레드의 등장

### 가상 스레드란?

가상 스레드는 JVM에서 관리하는 경량 스레드다. 운영체제 스레드와 직접 매핑되지 않고, 대신 **캐리어 스레드(Carrier Thread)** 라는 플랫폼 스레드 위에서 실행된다.

```java path=null start=null
// 가상 스레드 생성
Thread virtualThread = Thread.ofVirtual().start(() -> {
    System.out.println("Hello from virtual thread!");
});

// 또는 더 간단하게
Thread.startVirtualThread(() -> {
    System.out.println("Another virtual thread!");
});
```

### 가상 스레드의 핵심 특징

1. **극도로 경량**: 메모리 사용량이 수 KB 수준
2. **대량 생성 가능**: 수백만 개의 가상 스레드도 문제없이 생성 가능
3. **간단한 프로그래밍 모델**: 동기 스타일의 코드 작성 가능

## 실제 사용은?

### 동시 작업 처리

```java path=null start=null
public class VirtualThreadExample {
    public static void main(String[] args) throws InterruptedException {
        var executor = Executors.newVirtualThreadPerTaskExecutor();
        
        // 10,000개의 작업을 동시에 실행
        for (int i = 0; i < 10000; i++) {
            final int taskId = i;
            executor.submit(() -> {
                try {
                    // I/O 작업 시뮬레이션
                    Thread.sleep(Duration.ofSeconds(1));
                    System.out.println("Task " + taskId + " completed");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        executor.shutdown();
    }
}
```

## 기술적 심화 내용

### 가상 스레드의 내부 구조

가상 스레드는 **M:N 스레딩 모델**을 구현한다. 여기서 M개의 가상 스레드가 N개의 캐리어 스레드(플랫폼 스레드) 위에서 실행된다.

#### 핵심 컴포넌트

1. **VirtualThread**: 가상 스레드의 실제 구현체
2. **Carrier Thread**: 가상 스레드를 실행하는 플랫폼 스레드
3. **ForkJoinPool**: 기본 캐리어 스레드 풀 (기본적으로 CPU 코어 수만큼 생성)
4. **Continuation**: 가상 스레드의 실행 상태를 저장하는 객체

### 마운트/언마운트 메커니즘

가상 스레드의 가장 중요한 특징 중 하나는 **마운트(mount)** 와 **언마운트(unmount)** 메커니즘이다.

마운트
- 가상 스레드가 플랫폼 스레드에 연결되어 실행되는 상태
- 갓ㅏㅇ 스레드가 CPU에서 실제로 코드를 실행할 수 있는 상태
- JVM이 가상 스레드를 캐리어 스레드에 할당하는 과정

언마운트
- 가상 스레드가 플랫폼 스레드에서 분리된 상태
- 주로 블로킹 작업(I/O, sleep 등)이 발생할때 일어남
- 다른 가상 스레드가 해당 플랫폼 스레드를 사용할 수 있도록 양보


```java path=null start=null
// 가상 스레드가 블로킹 I/O를 만났을 때
public void performDatabaseQuery() {
    // 1. 가상 스레드가 캐리어 스레드에 마운트된 상태
    var connection = dataSource.getConnection();
    // 2. I/O 블로킹 발생 시 자동으로 언마운트
    var result = connection.executeQuery("SELECT * FROM users");
    // 3. I/O 완료 시 다시 마운트 (다른 캐리어 스레드일 수 있음)
    processResult(result);
}
```

### 스케줄링 알고리즘

가상 스레드는 **협력적 멀티태스킹(Cooperative Multitasking)** 을 사용:

1. **Blocking Operations**: I/O, sleep, wait 등에서 자동 언마운트
2. **CPU 집약적 작업**: yield point에서만 스케줄링 가능
3. **Work-Stealing**: ForkJoinPool의 work-stealing 알고리즘 활용
   > 그렇다면 Work-stealing 알고리즘은 무엇인가?
   > 1. 각 워커 스레드가 자신만의 작업 큐(deque)를 가짐
   > 2. 자신의 큐가 비어있으면 다른 스레드의 큐에서 작업을 훔쳐옴
   > 3. 효율적인 부하 분산과 CPU 활용도 극대화

### 메모리 관리 최적화

```java path=null start=null
// 가상 스레드의 스택 구조
// 플랫폼 스레드: 고정 크기 스택 (보통 1-2MB)
// 가상 스레드: 동적 크기 스택 (초기 수 KB, 필요시 확장)

public class StackMemoryComparison {
    public static void demonstrateMemoryUsage() {
        // 플랫폼 스레드: 2MB × 1000 = 2GB
        for (int i = 0; i < 1000; i++) {
            new Thread(() -> {
                deepRecursion(1000);
            }).start();
        }
        
        // 가상 스레드: ~10KB × 1000000 = ~10GB (하지만 실제로는 훨씬 효율적)
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 1000000; i++) {
                executor.submit(() -> {
                    shallowOperation();
                });
            }
        }
    }
}
```

## 성능 특성

### 처리량(Throughput) 비교

- **플랫폼 스레드**: 약 200-300 동시 연결 처리 시 성능 저하
- **가상 스레드**: 100,000개 이상의 동시 연결도 효율적으로 처리

### 레이턴시(Latency) 특성

- **낮은 생성 비용**: 플랫폼 스레드 대비 약 1000배 빠른 생성
- **효율적인 컨텍스트 스위칭**: JVM 내부에서 처리되므로 OS 레벨 스위칭 없음

## 제약사항과 주의사항

### Pinning 문제

가상 스레드가 캐리어 스레드에 "고정"되는 상황:

```java path=null start=null
// 문제가 되는 코드
synchronized (lock) {
    // 이 블록 내에서 I/O 작업을 하면 pinning 발생
    expensiveIoOperation(); // 가상 스레드가 언마운트되지 않음
}

// 해결책: ReentrantLock 사용
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    expensiveIoOperation(); // 정상적으로 언마운트 가능
} finally {
    lock.unlock();
}
```

### ThreadLocal 사용 주의

```java path=null start=null
// ThreadLocal을 많이 사용하는 경우 메모리 사용량 증가
ThreadLocal<ExpensiveObject> threadLocal = ThreadLocal.withInitial(ExpensiveObject::new);

// 가상 스레드가 수백만 개 생성되면 ExpensiveObject도 수백만 개 생성
// 대안: ScopedValue (Java 21+)
static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

ScopedValue.where(USER_ID, "user123")
    .run(() -> {
        // USER_ID.get()으로 값 접근 가능
        processRequest();
    });
```

## 실제 사용 사례

### 적합한 사용 사례

1. **I/O 집약적 애플리케이션**: 웹 서버, 마이크로서비스
2. **높은 동시성이 필요한 시스템**: 채팅 서버, 실시간 데이터 처리
3. **블로킹 API 래핑**: 기존 동기식 라이브러리 활용

### 부적합한 사용 사례

1. **CPU 집약적 작업**: 복잡한 연산, 암호화 등
2. **Fine-grained 병렬 처리**: 작은 단위의 병렬 작업
3. **실시간 시스템**: 예측 가능한 레이턴시가 필요한 경우