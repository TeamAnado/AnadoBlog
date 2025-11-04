# 자바의 컬렉션에 대해서

가장 많이 사용되는 ArrayList, HashMap, HashSet을 중심으로 시나리오 예제와 함께 공부한 내용을 정리해봤다. 

## 왜 인터페이스와 구현체를 분리했을까?

자바 컬렉션 프레임워크에서 `List`, `Set`, `Map`은 인터페이스고, `ArrayList`, `HashSet`, `HashMap`은 구현체다. 이렇게 복잡하게 만든 이유는 **유연성** 때문이다.

초기에 ArrayList로 개발했다가 나중에 성능 문제로 LinkedList로 교체해야 하는 상황을 생각해보자. 코드 전체에 `ArrayList` 타입을 명시했다면 모든 곳을 수정해야 하지만, `List` 인터페이스를 사용했다면 객체 생성 부분만 바꾸면 된다.

```java
// 나쁜 예: 구체 타입에 의존
ArrayList<String> names = new ArrayList<>();

// 좋은 예: 인터페이스에 의존
List<String> names = new ArrayList<>();
// 나중에 교체하기 쉽다
// List<String> names = new LinkedList<>();
```

이런 설계 철학은 SOLID 원칙 중 **의존성 역전 원칙(DIP)**과 일맥상통한다. 구체적인 구현이 아닌 추상화에 의존하라는 것.

---

## ArrayList: 가장 무난하지만 가장 오해받는 컬렉션

### 내부 구조와 성장 메커니즘

ArrayList를 "동적 배열"이라고 부르는데, 정확히 무슨 의미일까? 실제로는 내부에 고정 크기 배열(`Object[]`)을 가지고 있다. 요소를 추가할 때마다 배열이 자동으로 늘어나는 게 아니라, **용량이 부족하면 더 큰 배열을 새로 만들고 기존 데이터를 복사**하는 방식이다.

```java
// ArrayList 내부 동작 (간략화)
public class ArrayList<E> {
    private Object[] elementData;
    private int size;
    
    public boolean add(E e) {
        if (size == elementData.length) {
            // 새 배열 생성 (보통 1.5배 크기)
            elementData = Arrays.copyOf(elementData, newCapacity());
        }
        elementData[size++] = e;
        return true;
    }
}
```

### 대량 데이터 처리 시나리오

배치 작업에서 100만 건의 고객 데이터를 처리해야 하는 상황을 생각해보자. 기본 생성자로 ArrayList를 만들면 초기 용량이 10이므로, 100만 건을 담기 위해 수십 번의 재할당과 복사가 발생한다. 이는 불필요한 메모리 할당과 GC 부담을 야기한다.

```java
// 비효율: 기본 용량 10으로 시작, 계속 재할당
List<Customer> customers = new ArrayList<>();
for (Customer c : millionCustomers) {
    customers.add(c); // 여러 번 배열 재할당 발생
}

// 효율적: 초기 용량 지정으로 재할당 최소화
List<Customer> customers = new ArrayList<>(1_000_000);
for (Customer c : millionCustomers) {
    customers.add(c); // 재할당 없음
}
```

이렇게 간단한 변경만으로도 처리 시간과 메모리 사용량을 크게 줄일 수 있다. 특히 대량 데이터를 다룰 때는 초기 용량 지정이 필수다.

### 중간 삽입/삭제가 빈번한 시나리오

ArrayList는 **인덱스 접근은 O(1)로 빠르지만, 중간 삽입/삭제는 O(n)으로 느리다**. 이는 삽입/삭제 지점 이후의 모든 요소를 이동시켜야 하기 때문이다.

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));

// O(1) - 빠름
String item = list.get(2);

// O(n) - 느림! 뒤의 모든 요소를 한 칸씩 이동
list.remove(1); // "B" 삭제 후 C, D, E를 앞으로 이동
list.add(1, "X"); // X 삽입 후 C, D, E를 뒤로 이동
```

실시간 알림 시스템에서 새 알림을 리스트 앞쪽에 추가하고 오래된 알림을 중간에서 삭제하는 구조를 생각해보자. 사용자 수가 늘어나면 매번 O(n) 연산이 반복되어 성능이 급격히 저하된다. 이런 경우 LinkedList나 Deque 구조가 더 적합하다.

### 스레드 안전성 문제

ArrayList는 **동기화되지 않는다**. 멀티스레드 환경에서 여러 스레드가 동시에 접근하면 `ConcurrentModificationException`이 발생하거나 데이터가 손상될 수 있다.

```java
// 위험한 코드
List<String> list = new ArrayList<>();
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 100; i++) {
    executor.submit(() -> list.add("item")); // Race condition!
}
```

해결 방법:
1. `Collections.synchronizedList()` - 간단하지만 모든 메서드에 락이 걸려 성능 저하
2. `CopyOnWriteArrayList` - 읽기가 많고 쓰기가 적을 때 적합
3. 외부 동기화 - `synchronized` 블록 사용

대부분의 경우 **불변 리스트**를 사용하거나, 정말 필요한 경우에만 동시성 컬렉션을 사용하는 것이 안전하다.

---

## HashMap: 키-값 저장의 절대 강자

### 해시 테이블의 동작 원리

HashMap을 이해하려면 **해시 함수**를 먼저 알아야 한다. 키 객체를 받아서 정수(해시 코드)로 변환하고, 이를 배열 인덱스로 사용하는 방식이다.

```java
// HashMap 동작 원리 (단순화)
int hash = key.hashCode();
int index = hash % buckets.length;
buckets[index] = new Entry(key, value);
```

이론적으로는 O(1) 시간에 데이터를 찾을 수 있다. 하지만 **해시 충돌**이라는 함정이 있다.

### 해시 충돌과 자바 8의 개선

두 개의 서로 다른 키가 같은 해시 코드를 생성하면 같은 버킷에 저장되어야 한다. 자바 7까지는 **연결 리스트**로 관리했기 때문에 충돌이 많아지면 O(n)까지 성능이 저하되는 문제가 있었다.

자바 8부터는 영리한 최적화를 도입했다. 한 버킷에 8개 이상의 요소가 충돌하면 **레드-블랙 트리로 전환**한다. 이렇게 하면 최악의 경우에도 O(log n)을 보장할 수 있다.

```java
// Java 8+ HashMap 내부
static final int TREEIFY_THRESHOLD = 8;

// 충돌이 심하면 트리로 변환
if (binCount >= TREEIFY_THRESHOLD) {
    treeifyBin(tab, hash);
}
```

이 변경은 보안 측면에서도 중요하다. 이전에는 악의적으로 설계된 입력값으로 HashMap의 성능을 O(n)까지 떨어뜨리는 **해시 충돌 공격**이 가능했다.

### 커스텀 객체를 키로 사용할 때의 함정

HashMap을 사용할 때 가장 흔한 실수는 **커스텀 객체를 키로 사용하면서 hashCode()와 equals()를 재정의하지 않는 것**이다.

```java
class User {
    private String email;
    private String name;
    
    // hashCode()와 equals() 미구현!
}

// 문제 발생
Map<User, String> userMap = new HashMap<>();
User user1 = new User("john@example.com", "John");
userMap.put(user1, "Developer");

User user2 = new User("john@example.com", "John");
String role = userMap.get(user2); // null 반환!
```

user1과 user2는 논리적으로 같은 사용자지만, 기본 Object의 hashCode()와 equals()는 **객체의 메모리 주소**를 기준으로 비교하기 때문에 다른 객체로 인식된다.

```java
class User {
    private String email;
    private String name;
    
    @Override
    public int hashCode() {
        return Objects.hash(email, name);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof User)) return false;
        User other = (User) obj;
        return Objects.equals(email, other.email) 
            && Objects.equals(name, other.name);
    }
}
```

### 초기 용량과 로드 팩터 최적화

HashMap의 기본 초기 용량은 16이고, 로드 팩터는 0.75다. 로드 팩터란 "용량의 75%가 차면 크기를 두 배로 늘리겠다"는 의미다.

저장할 요소의 개수를 미리 안다면 적절한 초기 용량을 설정하여 재해싱 오버헤드를 피할 수 있다.

```java
// 비효율: 여러 번 재해싱 발생
Map<String, Data> cache = new HashMap<>();
for (int i = 0; i < 10000; i++) {
    cache.put("key" + i, new Data(i));
}

// 효율적: 재해싱 없음
int expectedSize = 10000;
Map<String, Data> cache = new HashMap<>((int) (expectedSize / 0.75 + 1));
for (int i = 0; i < 10000; i++) {
    cache.put("key" + i, new Data(i));
}
```

### 멀티스레드 환경의 위험성

멀티스레드 환경에서 HashMap을 사용하면 **데이터 손상**이나 심지어 **무한 루프**가 발생할 수 있다. 특히 자바 7의 경우 리사이징 중 무한 루프가 발생하는 버그가 유명했다.

```java
// 절대 금지!
Map<String, String> map = new HashMap<>();
// 여러 스레드가 동시에 put() 호출 → 위험
```

해결책으로는 `Collections.synchronizedMap()`보다 **ConcurrentHashMap**을 사용하는 것이 훨씬 낫다. ConcurrentHashMap은 내부적으로 여러 세그먼트로 나누어 락을 걸기 때문에, 서로 다른 세그먼트에 대한 동시 접근이 가능하여 성능이 월등히 좋다.

---

## HashSet: 중복 제거의 마법사

### HashMap의 옷을 입은 HashSet

재미있는 사실: HashSet은 내부적으로 **HashMap을 사용**한다. 값은 더미 객체(PRESENT)로 채우고, 키만 실제로 활용하는 방식이다.

```java
// HashSet 내부 (실제 코드)
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object();
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

이런 구조 덕분에 HashSet도 HashMap과 동일하게 O(1)의 add, remove, contains 성능을 제공한다.

### 대량 데이터 중복 제거 시나리오

HashSet이 진가를 발휘하는 순간은 **대량의 데이터에서 중복을 제거하거나 특정 값의 존재 여부를 빠르게 확인**할 때다.

100만 개의 이메일 주소 중 중복을 제거해야 하는 상황을 생각해보자. List를 사용하면 매번 contains()로 전체를 탐색해야 하므로 O(n²)의 시간이 소요된다. 반면 Set을 사용하면 O(n)에 해결된다.

```java
// 시나리오: 100만 개의 이메일 중 중복 제거
List<String> emails = fetchMillionEmails();

// 방법 1: List로 중복 체크 - O(n²), 매우 느림
List<String> unique1 = new ArrayList<>();
for (String email : emails) {
    if (!unique1.contains(email)) { // 매번 리스트 전체 탐색!
        unique1.add(email);
    }
}

// 방법 2: Set 사용 - O(n), 매우 빠름
Set<String> unique2 = new HashSet<>(emails);
```

실제로 테스트해보면 Set을 사용한 방법이 **수백 배에서 수천 배** 빠르다.

### 블랙리스트/화이트리스트 검증

스팸 필터링 시스템에서 50만 개의 블랙리스트 IP를 저장하고 들어오는 요청마다 검사해야 하는 경우를 생각해보자. ArrayList를 사용하면 매 요청마다 O(n) 탐색이 발생해 서버 성능이 심각하게 저하된다.

```java
// 비효율: O(n) 탐색, 서버 부하 높음
List<String> blacklist = loadBlacklist(); // 50만 개
boolean isBlocked = blacklist.contains(clientIp); // 최악의 경우 50만 번 비교

// 효율적: O(1) 탐색, 서버 부하 낮음
Set<String> blacklist = new HashSet<>(loadBlacklist());
boolean isBlocked = blacklist.contains(clientIp); // 평균 1번 비교
```

성능이 수천 배 향상되고, 서버 CPU 사용률도 극적으로 낮아진다.

### 순서가 필요할 때: LinkedHashSet

HashSet의 단점은 **순서를 보장하지 않는다**는 것이다. 삽입 순서를 유지하고 싶다면 LinkedHashSet을 사용하면 된다. 내부적으로 이중 연결 리스트를 추가로 유지하기 때문에 약간의 메모리 오버헤드가 있지만, 성능은 여전히 O(1)이다.

---

## LinkedList: 양방향 연결 리스트

### 언제 ArrayList 대신 LinkedList를 선택할까?

LinkedList는 이중 연결 리스트 구조로, 각 노드가 이전/다음 노드의 참조를 가진다. ArrayList와 달리 **중간 삽입/삭제가 O(1)**이지만, **인덱스 접근은 O(n)**이다.

큐(Queue)나 스택(Stack) 구현에 적합하다. 특히 양쪽 끝에서의 추가/삭제가 빈번한 경우 LinkedList가 유리하다. 예를 들어 작업 큐, 실행 취소/다시 실행 기능, LRU 캐시 같은 용도에 적합하다.

하지만 주의할 점이 있다. LinkedList는 각 노드마다 추가 메모리(참조 2개)를 사용하므로 **메모리 오버헤드**가 크다. 또한 노드들이 메모리상에 흩어져 있어 **캐시 지역성**이 떨어져 실제 성능은 이론보다 낮을 수 있다.

실무에서는 양방향 큐가 필요하면 **ArrayDeque**를 먼저 고려하는 것이 좋다. LinkedList보다 메모리 효율적이고 대부분의 경우 더 빠르다.

---

## TreeSet과 TreeMap: 정렬이 필요할 때

### 자동 정렬의 대가

TreeSet과 TreeMap은 **레드-블랙 트리** 기반 구조로, 요소들이 자동으로 정렬된 상태를 유지한다. 덕분에 정렬된 순회, 범위 검색, 최소/최대값 조회가 효율적이다.

하지만 대가가 있다. 삽입, 삭제, 검색 모두 **O(log n)**의 시간이 소요된다. HashSet/HashMap의 O(1)에 비해 느린 것이다.

### 언제 사용해야 할까?

TreeSet/TreeMap은 다음과 같은 경우에 적합하다:
- 데이터를 정렬된 상태로 유지해야 할 때
- 범위 검색이 필요할 때 (예: "10 이상 20 미만의 값")
- 최소/최대값을 자주 조회할 때
- 순위 기반 기능이 필요할 때

예를 들어 리더보드 시스템, 시계열 데이터 관리, 우선순위 기반 스케줄링 같은 경우에 유용하다.

```java
// 순위 시스템 예제
NavigableSet<Score> leaderboard = new TreeSet<>();
leaderboard.add(new Score("Alice", 100));
leaderboard.add(new Score("Bob", 85));
leaderboard.add(new Score("Charlie", 92));

// 자동으로 점수 기준 정렬됨
// 상위 N명 조회: leaderboard.descendingSet().stream().limit(10)
// 특정 범위 조회: leaderboard.subSet(from, to)
```

반면 단순히 정렬이 필요하다면, ArrayList에 담은 후 `Collections.sort()`를 사용하는 것이 더 효율적일 수 있다. 한 번만 정렬하면 되는 경우 O(n log n) 정렬이 여러 번의 O(log n) 삽입보다 빠를 수 있기 때문이다.

---

## Queue와 Deque: 작업 처리와 버퍼링

### ArrayDeque: 가장 빠른 양방향 큐

ArrayDeque는 크기 조정 가능한 배열 기반 양방향 큐로, LinkedList보다 대부분의 경우 **더 빠르고 메모리 효율적**이다. Stack이나 Queue가 필요하면 ArrayDeque를 사용하는 것이 좋다.

```java
// Stack으로 사용
Deque<String> stack = new ArrayDeque<>();
stack.push("first");
stack.push("second");
String top = stack.pop(); // "second"

// Queue로 사용
Deque<String> queue = new ArrayDeque<>();
queue.offer("first");
queue.offer("second");
String front = queue.poll(); // "first"
```

웹 크롤러의 URL 큐, 이벤트 처리 버퍼, BFS 탐색 같은 용도에 적합하다.

### PriorityQueue: 우선순위 기반 처리

PriorityQueue는 **힙(Heap)** 구조로, 요소들이 우선순위에 따라 처리된다. 삽입과 삭제가 O(log n)이다.

작업 스케줄링, Dijkstra 알고리즘, 이벤트 시뮬레이션 같은 경우에 유용하다. 예를 들어 여러 작업 중 마감 시간이 가장 임박한 작업을 먼저 처리해야 하는 상황에서 적합하다.

주의할 점은 PriorityQueue가 **반복자를 통한 순회 시 정렬을 보장하지 않는다**는 것이다. 정렬된 순서로 요소를 꺼내려면 `poll()`을 반복적으로 호출해야 한다.

### BlockingQueue: 생산자-소비자 패턴

멀티스레드 환경에서 생산자-소비자 패턴을 구현할 때는 **BlockingQueue**가 적합하다. ArrayBlockingQueue, LinkedBlockingQueue 등이 있다.

큐가 가득 차면 생산자가 자동으로 대기하고, 큐가 비어 있으면 소비자가 자동으로 대기한다. 이는 명시적인 동기화 없이도 스레드 안전한 작업 처리 파이프라인을 구축할 수 있게 해준다.

```java
BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(100);

// 생산자 스레드
taskQueue.put(new Task()); // 큐가 가득 차면 대기

// 소비자 스레드
Task task = taskQueue.take(); // 큐가 비어 있으면 대기
```

배치 처리, 비동기 로깅, 메시지 버퍼링 같은 경우에 매우 유용하다.

---

## ConcurrentHashMap: 동시성의 왕

### 왜 HashMap 대신 ConcurrentHashMap인가?

멀티스레드 환경에서 여러 스레드가 동시에 Map을 수정해야 하는 경우가 많다. 캐시, 세션 저장소, 통계 수집 등이 대표적이다.

`Collections.synchronizedMap()`은 모든 메서드에 락을 걸기 때문에 **동시성이 심각하게 제한**된다. 한 번에 하나의 스레드만 접근할 수 있어 병목이 발생한다.

ConcurrentHashMap은 **세그먼트 기반 락킹** (자바 8 이후로는 더 개선된 메커니즘)을 사용하여, 서로 다른 부분에 대한 동시 접근을 허용한다. 읽기 연산은 대부분 락 없이 수행되고, 쓰기 연산만 해당 버킷에 대해 락을 건다.

```java
// 웹 애플리케이션 세션 관리 예제
Map<String, Session> sessions = new ConcurrentHashMap<>();

// 여러 스레드가 동시에 안전하게 접근 가능
sessions.put(sessionId, new Session(user));
Session session = sessions.get(sessionId);
sessions.remove(expiredSessionId);
```

### 원자적 연산의 중요성

ConcurrentHashMap은 `putIfAbsent()`, `replace()`, `compute()` 같은 **원자적 복합 연산**을 제공한다. 이는 경쟁 조건(race condition)을 방지하는 데 필수적이다.

```java
// 잘못된 방식 - 경쟁 조건 발생 가능
if (!map.containsKey(key)) {
    map.put(key, value); // 다른 스레드가 중간에 끼어들 수 있음
}

// 올바른 방식 - 원자적 연산
map.putIfAbsent(key, value);

// 카운터 증가 (원자적)
map.compute(key, (k, v) -> v == null ? 1 : v + 1);
```

방문 횟수 카운팅, 재고 관리, 동시 접속자 수 추적 같은 경우에 매우 유용하다.

---

## 컬렉션 선택 가이드

### 의사결정 트리

**1. 순서가 있고 중복을 허용하는가?** → List
- 인덱스 접근이 많음: `ArrayList`
- 양 끝 삽입/삭제가 많음: `LinkedList` 또는 `ArrayDeque`
- 멀티스레드 + 읽기 많음: `CopyOnWriteArrayList`

**2. 순서는 중요하지 않고, 중복을 허용하지 않는가?** → Set
- 기본 선택: `HashSet`
- 삽입 순서 유지: `LinkedHashSet`
- 정렬 필요: `TreeSet`

**3. 키-값 쌍으로 저장하는가?** → Map
- 기본 선택: `HashMap`
- 삽입 순서 유지: `LinkedHashMap`
- 정렬 필요: `TreeMap`
- 멀티스레드: `ConcurrentHashMap`

**4. FIFO/LIFO 처리가 필요한가?** → Queue/Deque
- 기본 선택: `ArrayDeque`
- 우선순위: `PriorityQueue`
- 멀티스레드: `BlockingQueue` 구현체들

### 성능 최적화 체크리스트

- 저장할 데이터의 크기를 미리 알고 있다면 **초기 용량을 지정**했는가?
- 커스텀 객체를 키로 사용한다면 **hashCode()와 equals()를 재정의**했는가?
- 멀티스레드 환경에서 사용한다면 **적절한 동시성 컬렉션**을 선택했는가?
- 중간 삽입/삭제가 빈번한데 ArrayList를 사용하고 있지는 않은가?
- 단순히 존재 여부만 확인하는데 List를 사용하고 있지는 않은가?
- 자동 정렬이 불필요한데 TreeSet/TreeMap을 사용하고 있지는 않은가?

---

## 실전 안티패턴과 주의할 점 

### 안티패턴 1: null 체크 없이 맵 접근

HashMap에서 존재하지 않는 키로 get()을 호출하면 null이 반환된다. 이를 체크하지 않으면 NullPointerException이 발생한다.

```java
// 위험
Map<String, User> userMap = new HashMap<>();
String name = userMap.get("unknown").getName(); // NPE!

// 안전
User user = userMap.get("unknown");
if (user != null) {
    String name = user.getName();
}

// 더 나은 방법
String name = userMap.getOrDefault("unknown", defaultUser).getName();
```

### 안티패턴 2: 반복 중 컬렉션 수정

for-each 루프나 Iterator를 사용하는 중에 컬렉션을 직접 수정하면 `ConcurrentModificationException`이 발생한다.

```java
// 잘못된 방법
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
for (String item : list) {
    if (item.equals("B")) {
        list.remove(item); // 예외 발생!
    }
}

// 올바른 방법 1: Iterator 사용
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("B")) {
        it.remove();
    }
}

// 올바른 방법 2: removeIf (Java 8+)
list.removeIf(item -> item.equals("B"));
```

### 안티패턴 3: 불필요한 Boxing/Unboxing

기본 타입을 컬렉션에 저장하면 자동으로 박싱이 발생한다. 대량의 기본 타입 데이터를 다룰 때는 성능과 메모리 문제가 생긴다.

```java
// 비효율적: Integer 객체 100만 개 생성
List<Integer> numbers = new ArrayList<>();
for (int i = 0; i < 1_000_000; i++) {
    numbers.add(i); // boxing 발생
}

// 효율적: 기본 타입 배열 또는 IntStream 사용
int[] numbers = new int[1_000_000];
for (int i = 0; i < 1_000_000; i++) {
    numbers[i] = i;
}
```

### 안티패턴 4: 컬렉션을 API 경계에 그대로 노출

가변 컬렉션을 public 메서드의 반환값으로 직접 노출하면, 호출자가 내부 상태를 마음대로 변경할 수 있다.

```java
// 위험
public class Service {
    private List<String> items = new ArrayList<>();
    
    public List<String> getItems() {
        return items; // 외부에서 수정 가능!
    }
}

// 안전
public List<String> getItems() {
    return Collections.unmodifiableList(items);
}

// 더 나은 방법
public List<String> getItems() {
    return List.copyOf(items); // 불변 복사본 반환
}
```

---

## 불변 컬렉션의 힘

### 왜 불변이 중요한가?

불변 컬렉션은 한 번 생성되면 수정할 수 없다. 이는 여러 장점을 제공한다:
- **스레드 안전**: 동기화 없이도 여러 스레드에서 안전하게 공유
- **버그 방지**: 의도치 않은 수정을 컴파일 타임 또는 런타임에 차단
- **성능**: 불변성을 전제로 최적화 가능
- **캐싱**: 안전하게 캐시하고 재사용 가능

### 팩토리 메서드

```java
List<String> list = List.of("A", "B", "C");
Set<String> set = Set.of("A", "B", "C");
Map<String, Integer> map = Map.of("A", 1, "B", 2);

// 수정 시도하면 UnsupportedOperationException
list.add("D"); // 예외 발생
```

API 설계 시 가능한 한 불변 컬렉션을 사용하면 코드의 안정성과 예측 가능성이 크게 향상된다.

---

## Stream API와 컬렉션

### 선언적 데이터 처리

Java 8의 Stream API는 컬렉션 처리를 선언적으로 만들어준다. 필터링, 변환, 집계가 간결해진다.

```java
List<User> users = fetchUsers();

// 명령형 스타일
List<String> activeEmails = new ArrayList<>();
for (User user : users) {
    if (user.isActive()) {
        activeEmails.add(user.getEmail());
    }
}

// 선언형 스타일
List<String> activeEmails = users.stream()
    .filter(User::isActive)
    .map(User::getEmail)
    .collect(Collectors.toList());
```

### 적절한 Collector 선택

Stream API는 다양한 Collector를 제공한다. 상황에 맞는 것을 선택해야 한다.

```java
// List로 수집
List<String> list = stream.collect(Collectors.toList());

// Set으로 수집 (중복 자동 제거)
Set<String> set = stream.collect(Collectors.toSet());

// Map으로 수집
Map<String, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getEmail, Function.identity()));

// 그룹핑
Map<String, List<User>> usersByDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment));
```

### 병렬 스트림 주의사항

`parallelStream()`은 멀티코어를 활용할 수 있지만, 항상 빠른 것은 아니다. 작은 데이터셋이나 가벼운 연산에서는 오히려 오버헤드가 더 클 수 있다.

병렬 스트림은 다음 조건에서 고려해볼 만하다:
- 데이터가 충분히 많음 (보통 수천 개 이상)
- 각 요소의 처리가 무거움
- 순서가 중요하지 않음
- 스레드 안전한 연산만 사용

---

## 핵심 결론

- **ArrayList**: 인덱스 접근이 많을 때, 대량 데이터는 초기 용량 지정
- **HashMap**: 키-값 저장의 기본, 커스텀 객체는 hashCode/equals 필수
- **HashSet**: 중복 제거와 빠른 검색, 내부는 HashMap 활용
- **LinkedList**: 중간 삽입/삭제가 많을 때, 하지만 ArrayDeque도 고려
- **TreeSet/TreeMap**: 정렬이 필요할 때, O(log n) 비용 감수
- **ConcurrentHashMap**: 멀티스레드 환경의 기본 선택
- **불변 컬렉션**: 가능하면 불변으로, 안정성과 성능 향상

상황에 맞는 컬렉션을 선택하는 것만으로도 성능이 몇 배에서 수천 배까지 차이 날 수 있다. 왜 이 컬렉션을 선택했는지, 과연 올바른 선택이었는지? 항상 의심해보는 습관을 가져보자.
