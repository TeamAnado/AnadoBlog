
## Fail-Fast vs Fail-Safe Iterator

자바 컬렉션은 `fail-safe`와 `fail-fast` 2가지 Iterator 동작 방식이 있다. `fail-safe`는 컬렉션이 순환 도중에 변경이 가능한 경우이고, `fail-fast`는 컬렉션이 순환 도중에 변경이 불가능한 경우이다. `fail-fast`의 경우 비동기적인 작업 중에 동시적인 변경을 보장하지 못한다. 따라서 thread-safe하지 못한다는 소리이다.

java의 컬렉션은 modCount 라는 내부의 counter를 운용합니다. Collection에 item이 추가되거나 제거될 때마다 이 modCount 라 불리는 카운터가 증가합니다. iterating 중에 next()를 호출 할 때 마다 현 상황의 modCount 값을 초기의 값과 비교를 하는데요, 만약 거기에서 조금이라도 다르다면 즉시 ConcurrentModificationException을 throw 하고 전체 작업을 중단합니다.

`fail-fast` Iterator는 **비동기 컬렉션** (예: `ArrayList`, `HashMap`)에서 생성됩니다.
- 특징: 내부의 **`modCount`**를 검사하여 외부 수정 시 `ConcurrentModificationException`을 발생시킨다.
`fail-safe` Iterator는 **동시성 컬렉션** (예: `ConcurrentHashMap`, `CopyOnWriteArrayList`)에서 생성된다.
- 특징: 반복 시 **컬렉션의 복사본(Snapshot)**을 사용하거나 **원자적(Atomic) 연산**을 수행하므로, 반복 중 원본이 변경되어도 예외가 발생하지 않는다.

예를 들어 ArrayList의 경우, 순환을 할 때, 다른 쓰레드가 해당 컬렉션을 변경해서는 안된다. 만약 Iterator가 순환을 하고 있는도중 새로운 요소의 추가, 삭제 시도를 감지한다면, ArrayList는 `fail-fast`이므로 `ConccurrentModificationException`을 던진다. 

`Iterator.remove()`에서는 예외를 던지지 않고 컬렉션의 `remove()`에서만 예외를 던진다. 이는 `fail-fast`의 동작 원리 때문이다. `ArrayList`와 같은 `fail-fast` 컬렉션(비동기화된 컬렉션)은 내부적으로 `modCount` (Modification Count)라는 변수를 사용하여 컬렉션의 구조적 변경 횟수를 추적한다. 

1. Iterator 생성 시 `Iterator`가 생성될 때, 컬렉션의 현재 `modCount` 값을 복사해 둔다.
2. 반복 진행 시 `Iterator`가 `next()`를 호출할 때마다, 자신이 복사해 둔 값과 컬렉션의 현재 `modCount`를 비교한다.
3. 불일치 시 두 값이 다르면 (**외부에서 변경이 있었다고 판단**), 즉시 **`ConcurrentModificationException`**을 발생시킨다.

### 컬렉션의 `remove()`와 `Iterator.remove()`의 차이

`modCount`를 관리하는 방식이 다르다!

#### 컬렉션의 `remove()` (예외 발생 조건)

- `ArrayList.remove(Object o)`와 같이 **컬렉션 객체에 직접** 삭제를 요청하는 행위는 **Iterator의 관점에서는 '외부의 동시적 수정'**으로 간주됩니다.
- 컬렉션은 요소를 삭제하고 **`modCount`를 증가**시킵니다.
- 하지만 **`Iterator`는 자신의 복사된 `modCount`를 업데이트할 기회가 없으므로**, 다음에 `next()`를 호출할 때 `modCount` 불일치를 감지하고 **`ConcurrentModificationException`**을 던집니다.
- **"컬렉션의 `remove()`에서만 예외를 던진다"**는 것은 **Iterator 외부의 수정**을 의미합니다.
    

#### `Iterator.remove()` (예외를 피하는 방법)
- 이 메서드는 `Iterator`가 **자신의 제어 하에** 컬렉션을 수정하는 **유일하게 안전한 방법**입니다.
- **핵심:** `Iterator.remove()`는 요소를 제거하는 과정에서 컬렉션의 `modCount`를 증가시킴과 동시에, **`Iterator` 자신이 복사해 둔 내부 `modCount` 값도 함께 업데이트**합니다.
- 따라서 다음에 `next()`를 호출하여 비교해도 두 `modCount` 값은 일치하므로 **예외가 발생하지 않습니다.**
- **"`Iterator.remove()`에서는 예외를 던지 않고"**는 **Iterator 자신의 통제된 내부 수정**을 의미합니다.

itr.remove()는 요소를 제거한 후 **Iterator 자신이 내부 `modCount` 상태를 즉시 업데이트**하기 때문에 오류를 발생시키지 않습니다. 반면, `al.remove()`와 같이 **컬렉션에 직접 접근**하는 것은 **Iterator 외부의 수정**으로 간주되어 `modCount` 불일치를 유발하고 예외를 던집니다.

그렇다면 Iterator.remove()와 fail-safe한 컬렉션의 remove는 어떻게 다를까? 이 둘은 컬렉션을 수정한다는 점만 같을 뿐, 스레드 안전성(Thread-Safety)을 보장하는 방식과 예외를 회피하는 메커니즘이 완전히 다르다.

`Iterator.remove()`는 컬렉션 자체를 `fail-safe`하게 만들지는 못하며, 단순히 **반복자 자신을 통과하는 일회성 수정**만 허용합니다. 따라서 thread-safe하지 않다. 반복자(`Iterator`)가 요소를 삭제한 후, **자신의 내부 상태(`modCount`)를 직접 업데이트**하여 다음에 진행될 `fail-fast` 검사를 **능동적으로 회피**합니다.

### Fast-Fail을 예방하기 위해 복사본을 만드는 방법과 동기화를 보장하는 방법

CopyOnWriteArrayList로 배열을 복사하여 사용하면, 순환 도중에 추가 혹은 삭제를 하여로 오류를 던지지 않습니다.

```java
class FailSafe { 
	public static void main(String args[]) { 
		CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<Integer>(new Integer[] { 1, 3, 5, 8 }); 
		Iterator itr = list.iterator();

		while (itr.hasNext()) { 
			Integer no = (Integer)itr.next();
			System.out.println(no);
			if (no == 8) 
			// This will not print, 
			// hence it has created separate copy 
			list.add(14); 
		} 
	} 
}
```


동기화를 보장하는 컬렉션 사용

```java
public class FailSafeItr { 
	public static void main(String[] args) { 
		// Creating a ConcurrentHashMap 
		ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<String, Integer>(); 
		map.put("ONE", 1); 
		map.put("TWO", 2); 
		map.put("THREE", 3); 
		map.put("FOUR", 4); 
	
		// Getting an Iterator from map 
		Iterator it = map.keySet().iterator(); 
		while (it.hasNext()) { 
			String key = (String)it.next(); 
			System.out.println(key + " : " + map.get(key)); 
			// This will reflect in iterator. 
			// Hence, it has not created separate copy 
			map.put("SEVEN", 7); 
		} 
	} 
}

```

순환(iteration) 도중 컬렉션이나 리스트의 remove() 를 사용하면, CoccurentModificationException을 던집니다. 따라서 컬렉션 요소를 제거할 때는 순환자(Iterator) remove() 를 사용해야 합니다.


### 장단점

fail-safe
 - Iterator가 실제 콜렉션 대신 복제본에서 작업하고 있기 때문에 콜렉션에서 업데이트 된 데이터를 반환하지 않는다는 것
 - 시간과 메모리와 관련하여 콜렉션의 사본을 작성하는 오버 헤드
