# ☕ Java 데이터 컬렉션 (JCF) 완벽 정리

Java 데이터 컬렉션(Java Collections Framework, JCF)은 여러 개의 데이터를 효과적으로 저장, 관리, 조작할 수 있도록 표준화된 방법을 제공하는 클래스 및 인터페이스의 집합입니다.

# 1. 데이터 컬렉션(JCF)이란 무엇인가요?

JCF는 간단히 말해 **'데이터를 담는 표준 그릇(컨테이너)'**입니다. 개발자가 직접 자료구조를 만들 필요 없이, 이미 잘 만들어지고 최적화된 다양한 '그릇'들을 가져다 쓸 수 있게 해줍니다.

주요 구성 요소는 다음과 같습니다.

인터페이스 (Interfaces): 컬렉션이 가져야 할 기본 동작(메서드)을 정의한 '설계도'.

예: List, Set, Map

구현 클래스 (Implementations): 인터페이스(설계도)를 실제로 구현한 '제품'.

예: ArrayList, HashSet, HashMap

알고리즘 (Algorithms): 데이터 정렬(sort), 검색(binarySearch) 등 유용한 기능을 제공하는 유틸리티.

예: Collections 클래스

# 2. 데이터 컬렉션을 왜 써야 하나요?

기본적인 **배열(Array)**은 다음과 같은 한계가 있습니다.

고정된 크기: 한 번 만들면 크기 변경이 불가능합니다.

불편한 관리: 데이터 삽입/삭제 시 요소를 직접 이동시켜야 합니다.

제한된 기능: 정렬, 검색 등을 직접 구현해야 합니다.

컬렉션(JCF)은 이러한 문제들을 해결해 줍니다.

✅ 동적 크기 조절: 데이터 양에 따라 공간이 자동으로 늘어나거나 줄어듭니다.

✅ 편리한 API: add(), remove(), contains() 등 다양한 관리 메서드를 제공합니다.

✅ 다양한 자료구조: 목적에 맞는 최적의 '그릇'(List, Set, Map)을 선택할 수 있습니다.

✅ 성능 향상: 고도로 최적화된 자료구조를 사용하여 성능이 보장됩니다.

✅ 표준화: 유연하고 재사용성 높은 코드를 작성할 수 있습니다.

# 3. 데이터 컬렉션의 예시

JCF의 가장 핵심적인 3가지 인터페이스 예시입니다.

## 🔹 1. List 인터페이스

특징: 순서가 있으며, 데이터 중복을 허용합니다. (목록)

주요 구현: ArrayList, LinkedList

### Java

// List 예시: ArrayList
List<String> fruits = new ArrayList<>();
fruits.add("사과");
fruits.add("바나나");
fruits.add("사과"); // 중복 허용

// 순서(인덱스)로 조회
System.out.println(fruits.get(0)); // 출력: 사과
System.out.println(fruits.size()); // 출력: 3

## 🔹 2. Set 인터페이스

특징: 순서를 보장하지 않으며, 데이터 중복을 허용하지 않습니다. (집합)

주요 구현: HashSet, TreeSet

### Java

// Set 예시: HashSet
Set<String> uniqueColors = new HashSet<>();
uniqueColors.add("빨강");
uniqueColors.add("파랑");
uniqueColors.add("빨강"); // 중복 무시

System.out.println(uniqueColors.size()); // 출력: 2
System.out.println(uniqueColors.contains("파랑")); // 출력: true

## 🔹 3. Map 인터페이스

특징: **키(Key)**와 **값(Value)**의 쌍으로 데이터를 저장합니다. 키는 중복될 수 없습니다. (사전, 전화번호부)

주요 구현: HashMap, TreeMap

### Java

// Map 예시: HashMap
Map<String, Integer> userAgeMap = new HashMap<>();
userAgeMap.put("홍길동", 30);
userAgeMap.put("이순신", 45);
userAgeMap.put("홍길동", 31); // 키 중복 -> 값이 31로 덮어써짐

// 키로 값 조회
System.out.println(userAgeMap.get("이순신")); // 출력: 45
System.out.println(userAgeMap.size()); // 출력: 2

# 4. 파생되어 알아야 할 것들

## 🔹 1. 제네릭 (Generics, <E>)

개념: 컬렉션에 저장할 데이터 타입을 미리 지정하는 기능입니다. (예: List<String>)

이유:

타입 안정성: 컴파일 시점에 잘못된 타입이 들어오는 것을 막아줍니다.

형 변환 불필요: 데이터를 꺼낼 때마다 귀찮은 형 변환((String) obj)이 필요 없습니다.

### Java

// Good: 제네릭 사용
List<String> names = new ArrayList<>();
names.add("홍길동");
// names.add(123); // 컴파일 에러!

String name = names.get(0); // 형 변환 필요 없음

// Bad: 제네릭 미사용 (Raw Type)
List list = new ArrayList();
list.add("홍길동");
list.add(123); // 에러가 안 나지만, 나중에 문제를 일으킴

String name2 = (String) list.get(0); // 형 변환 필수
// String name3 = (String) list.get(1); // 런타임 에러! (ClassCastException)

## 🔹 2. Collections 유틸리티 클래스

개념: 컬렉션을 다루는 데 유용한 정적(static) 메서드들의 모음입니다.

주요 기능: 정렬, 셔플(섞기), 최대/최소값 찾기 등.

### Java

List<Integer> numbers = new ArrayList<>(List.of(5, 1, 10, 3));

// 1. 정렬 (오름차순)
Collections.sort(numbers);
System.out.println("정렬: " + numbers); // [1, 3, 5, 10]

// 2. 섞기 (셔플)
Collections.shuffle(numbers);
System.out.println("셔플: " + numbers); // (실행 시마다 다름)

## 🔹 3. 스트림 API (Stream API, Java 8+)

개념: 컬렉션 데이터를 함수형으로 간결하게 처리(필터링, 변환, 집계)하는 기능입니다.

이유: 복잡한 for문과 if문 없이도 가독성 높은 데이터 처리가 가능합니다.

### Java

List<String> names = List.of("Apple", "Banana", "Avocado", "Blueberry");

// [목표] 'A'로 시작하는 이름만 대문자로 바꿔서 리스트로 만들기

// 스트림 API 사용
List<String> result = names.stream()
.filter(name -> name.startsWith("A")) // 필터링
.map(name -> name.toUpperCase()) // 변환
.collect(Collectors.toList()); // 수집

System.out.println("스트림 결과: " + result); // [APPLE, AVOCADO]

# 5. 데이터 컬렉션 사용 시 주의할 점 ⚠️

적절한 컬렉션 선택하기 (성능)

조회가 많으면 ➔ ArrayList

중간 삽입/삭제가 매우 빈번하면 ➔ LinkedList (대부분은 ArrayList가 더 유리함)

중복 제거만 필요하면 ➔ HashSet (가장 빠름)

정렬된 상태로 중복 제거 ➔ TreeSet

빠른 검색 (Key-Value) ➔ HashMap (순서 X)

정렬된 검색 (Key-Value) ➔ TreeMap (키 기준 정렬)

인터페이스 타입으로 선언하기 (유연성)

좋은 예: List<String> list = new ArrayList<>();

나쁜 예: ArrayList<String> list = new ArrayList<>();

이유: '좋은 예'처럼 선언하면, 나중에 new LinkedList<>()로 쉽게 구현체를 바꿀 수 있어 유연합니다.

제네릭(Generics)은 필수로 사용하기

타입 안정성을 위해 List<String>처럼 항상 제네릭을 명시해야 합니다.

Null 값 처리 주의

HashMap, ArrayList 등은 null을 허용합니다.

TreeMap, TreeSet 등 정렬이 필요한 컬렉션은 null을 넣으면 NullPointerException이 발생할 수 있습니다.

ConcurrentHashMap 등 동시성 컬렉션은 null을 허용하지 않습니다.

동시성 문제 (Thread-Safety)

ArrayList, HashMap 등 기본 컬렉션은 Thread-safe 하지 않습니다.

여러 스레드가 동시에 접근해야 한다면, java.util.concurrent 패키지의 동시성 컬렉션(예: ConcurrentHashMap, CopyOnWriteArrayList)을 사용해야 합니다.
