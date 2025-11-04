## 자바 데이터 컬렉션??

`int[] array = new int[10];`

위와 같이 가장 기본적인 배열(Array)은 처음에 크기를 정해야 하고, 중간에 데이터를 삽입하거나 삭제하는 것이 매우 번거롭습니다.
이런 불편함을 해결하고, 데이터를 훨씬 더 효율적이고 유연하게 다룰 수 있게 해주는 표준 규격이 바로 **Java Collections Framework (JCF)** 입니다.

## 왜 써야하죠??

> 편하고, 빠르고, 표준화되어 있기 때문입니다.

- **편의성** : 데이터 추가(add), 삭제(remove), 검색(contains) 등 데이터를 다루는 데 필요한 수많은 기능을 미리 다 만들어져 있습니다.

- **성능** : 수많은 전문가가 최적화해 둔, 검증된 자료구조(Data Structure)를 골라 쓰기만 하면 됩니다.

- **표준화** : 데이터의 '특징'에 따라 List, Set, Map이라는 표준 인터페이스를 제공합니다. 덕분에 코드의 일관성과 재사용성이 좋아집니다.

## 컬렉션 종류

많은 인터페이스 중 핵심 3가지만 설명하겠습니다.

### `List<E>` (리스트)

- **특징**: 순서가 있고(Ordered), 데이터 중복을 허용합니다.
- **주요 구현체**

  - ArrayList: (가장 많이 씀!) 내부적으로 배열을 사용합니다. 데이터 조회(읽기)가 매우 빠릅니다. 하지만 데이터 추가/삭제나 중간에 끼워 넣을 때 느리다는 단점이 있습니다.

  - LinkedList: 내부적으로 노드들을 연결합니다. 데이터 추가/삭제가 매우 빠릅니다. 하지만 특정 순서의 데이터를 찾는 조회 속도는 느립니다.

### `Set<E>` (집합)

- **특징**: 순서를 보장하지 않고, 데이터 중복을 허용하지 않습니다.

- **주요 구현체**

  - HashSet: 가장 빠르고 일반적인 Set입니다. 순서를 전혀 보장하지 않습니다. (해시(Hash) 기반)

  - TreeSet: 데이터를 자동으로 정렬된 상태로 저장합니다. (트리(Tree) 기반)

  - LinkedHashSet: 입력된 순서대로 데이터를 유지합니다.

### `Map<K, V>` (맵)

- **특징** : Key(키)와 Value(값)를 한 쌍으로 저장합니다. Key는 중복될 수 없습니다.

- **주요 구현체**

  - HashMap: 가장 빠르고 일반적인 Map입니다. 순서를 보장하지 않습니다. (Key 조회가 O(1)에 가까움)

  - TreeMap: Key를 기준으로 자동으로 정렬된 상태로 Map을 유지합니다.

  - LinkedHashMap: 입력된 순서대로 Key-Value 쌍을 유지합니다.

### 그래서 언제 뭘 쓰는 게 좋죠?

| 상황                                                   | 추천 컬렉션                   | 이유                                       |
| ------------------------------------------------------ | ----------------------------- | ------------------------------------------ |
| 순서대로 데이터를 저장하고 자주 읽는다.                | ArrayList                     | 인덱스 기반 조회가 가장 빠릅니다.          |
| 데이터 중복을 제거해야 한다. (순서 무관)               | HashSet                       | 중복을 허용하지 않고 성능이 가장 좋습니다. |
| Key-Value 쌍으로 데이터를 저장하고 빠르게 찾아야 한다. | HashMap                       | Key를 통한 Value 검색 속도가 압도적입니다. |
| 데이터를 자주 추가/삭제해야 한다. (!! 중간에)          | LinkedList                    | 데이터 추가/삭제 시 오버헤드가 적습니다.   |
| 저장된 데이터를 자동 정렬된 상태로 유지해야 한다.      | TreeSet / TreeMap             | 데이터를 넣는 즉시 정렬 상태가 유지됩니다. |
| 넣은 순서대로 데이터를 관리해야 한다.                  | LinkedHashSet / LinkedHashMap | Set이나 Map에 순서가 필요할 때 사용합니다. |
| 작업을 순서대로 처리해야 한다. (대기열)                | Queue (LinkedList)            | FIFO 정책이 필요할 때 사용합니다.          |

## 좀 더 들어가 보면?

위의 내용에서 더 학습하면 좋을 내용을 가져와 봤습니다.

### Collections 유틸 클래스

Collection이 인터페이스라면, Collection's'는 컬렉션을 다루는 만능 도구(Utility) 클래스입니다. 모든 메소드가 static으로 제공됩니다.

- `sort(List<T> list)`: 리스트를 정렬합니다.

- `reverse(List<?> list)`: 리스트의 순서를 뒤집습니다.

- `shuffle(List<?> list)`: 리스트의 요소를 무작위로 섞습니다.

- `binarySearch(List<?> list, T key)`: 정렬된 리스트에서 이진 탐색으로 요소를 찾습니다.

- `unmodifiableList(List<? extends T> list)` : 리스트를 '읽기 전용'으로 만듭니다. (불변 객체)

### Comparable vs Comparator

`sort()`나 TreeSet/TreeMap을 쓸 때 대체 뭘 기준으로 정렬하라는 거지? 라는 궁금증이 생겼습니다.

#### Comparable<T> (비교 가능한)

- "나 스스로 정렬하는 기준을 갖고 있다." (기본 정렬)

- 객체(클래스)가 직접 compareTo() 메소드를 구현합니다.

- ex) String이나 Integer는 이미 Comparable이 구현되어 있어 정렬이 됩니다.

#### Comparator<T> (비교자)

- "제3자인 '심판'이 정렬 기준을 정해준다." (외부 정렬)

- `compare()` 메소드를 가진 별도의 클래스(혹은 람다식)를 만듭니다.

- ex) User 객체를 '나이순'으로도 정렬하고, '이름순'으로도 정렬하는 등 다양한 정렬 기준이 필요할 때 사용합니다.

### HashMap은 어떻게 그렇게 빠를까?

HashMap이 Key를 통해 Value를 $O(1)$(에 가깝게) 찾아내는 비결은 **해싱(Hashing)** 입니다.

1. `key.hashCode()`를 호출해 고유한 정수 값(해시코드)을 얻습니다.
2. 이 해시코드를 내부 배열(Bucket)의 **인덱스(Index)** 로 변환합니다.
3. 해당 인덱스에 Value를 바로 저장하거나 꺼내옵니다.

- hashCode()와 equals(): HashMap은 Key의 동등성을 이 두 메소드로 판단합니다.
  hashCode()가 같다면, 그 다음 equals()로 '진짜 같은지' 재확인합니다.

- **해시 충돌 (Hash Collision)** : 만약 서로 다른 Key가 같은 인덱스를 가리키면?
  - Java 7까지: LinkedList로 데이터를 줄줄이 연결합니다. (Separate Chaining)
  - Java 8부터: 데이터가 8개 이상 쌓이면 Red-Black Tree 구조로 변경하여, 최악의 경우에도 $O(logN)$의 검색 성능을 보장합니다

### 스트림 (Stream API)

Java 8부터 등장한 데이터 처리를 아주 깔끔하게 도와줍니다.

컬렉션의 데이터들을 **'흐름(Stream)'** 으로 보고, 이 흐름을 **'선언적'** 으로 처리합니다.

```java
// [Before] Java 7
List<String> names = new ArrayList<>();
for (User user : userList) {
    if (user.getAge() > 20) {
        names.add(user.getName());
    }
}

// [After] Java 8 Stream
List<String> names = userList.stream() // 1. 스트림 생성
                             .filter(user -> user.getAge() > 20) // 2. 중간 연산 (필터링)
                             .map(User::getName) // 3. 중간 연산 (변환)
                             .collect(Collectors.toList()); // 4. 최종 연산 (수집)
```

코드가 간결해지고, 데이터를 어떻게 처리할지(How)가 아닌 무엇을 할지(What)에 집중할 수 있게 됩니다.
