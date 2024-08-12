# 아이템 45. 불필요한 객체 생성을 피하라

객체의 생성은 언제나 비용이 들고 값 비싼 비용이 드는 객체 생성이 일어날 수 도 있다.

그러니 불필요한 객체 생성은 피하는 것이 최적화의 관점에서 좋다.

## 객체 생성 비용은 항상 클까?
어떤 객체를 wrap하면 크게 3가지 비용이 발생한다.
1. 객체는 더 많은 용량을 차지
2. 요소가 캡슐화되어 있으면, 접근에 추가적인 함수 호출 필요 -> 함수 처리 비용은 굉장히 값 싸지만, 티끌 모아 태산
3. 객체는 생성되어야 한다. -> 메모리 할당, 레퍼런스 만드는 작업.. ->  적은 비용이지만 티끌 모아 태산

## 객체 선언
매 순간 객체를 생성하지 않고, 객체를 재사용하는 간단한 방법은 싱글톤을 사용하는 것
```kt
sealed class LinkedList<T>

class Node<T>( 
	val head: T, 
	val tail: LinkedList<T> 
): LinkedList<T>()

class Empty<T>: LinkedList<T>()

val list1: LinkedList<Int> = Node(1, Node(2, Node, Empty()) 
val list2: LinkedList<Int> = Node(3, Node(4, Node, Empty()) // Empty() 인스턴스를 매번 만들어야 함

object Empty : LinkedList<Nothing>() 

// 사용
val list1: LinkedList<Int> = Node(1, Node(2, Empty))
val list2: LinkedList<Int> = Node(2, Node(2, Empty)) // 하나의 Empty 인스턴스를 재활용함
```

## 캐시 활용하는 팩토리 함수
객체 생성시 팩토리 함수를 사용하는 경우가 있음

팩토리 함수는 캐시를 가질 수 있어서 항상 같은 객체를 리턴할 수 있음

실제 stdlib의 `emptyList`는 이를 활용해 구현됨
```kt
fun <T> List<T> emtpyList() {
    return EMPTY_LIST
}
```
- 객체 생성이 무겁거나, 동시에 여러 mutable 객체를 사용하는 경우 객체 풀을 사용하는 것이 좋음
- 모든 순수 함수는 캐싱을 활용할 수 있음 -> 메모이제이션
```kt
private val FIB_CACHE = mutableMapOf<String, Connection>()

fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
		if (n <=1) BigInteger.ONE else fib(n - 1) + fib(n - 2)
}
```
이렇게 구현하면 추가적인 계산 없이 값을 구함 -> 효율성 증가, 하지만 메모리를 더 사용함
이럴 때, SoftReference를 사용하면 필요한 경우만 가비지 컬렉터가 메모리를 해제

## 무거운 객체 외부 스코프로 보내기

컬렉션 처리에서 이뤄지는 무거운 연산은 컬렉션 함수 내부에서 외부로 뺴는 것이 좋다.

최댓값의 수를 세는 확장 함수의 경우 아래 코드와 같다.
```kt
fun <T: Comparable<T>> Iterable<T>.countMax(): Int =
		count { it == this.max() }

fun <T: Comparable<T>> Iterable<T>.countMax(): Int =
		val max = this.max()
		return count { it == max }
}
```
max 값을 매번 찾지 않고 한번 찾아놓고 이를 활용해 성능 향상, 가독성 향상

## 지연 초기화
A클래스에 B, C, D라는 무거운 인스턴스가 필요하다고 가정해보자.
```kt
class A {
	val b = B()
	val c = C()
	val d = D()
}
```
A 객체를 생성하는 것은 굉장히 무거울 것이다. 이를 아래처럼 바꿔보자.
```kt
class A {
	val b by lazy { B() }
	val c by lazy { C() }
	val d by lazy { D() }
}
```
이 경우 A 클래스 객체를 생성해도 b, c, d는 생성하지 않아도 괜찮아 A의 생성이 굉장히 가벼워집니다.

하지만, 첫 호출 시 응답이 오래걸려 백엔드에서 좋지 않을 수 있고..

성능 테스트가 복잡해 집니다.

상황에 맞게 사용합시다.

## 원시타입 자료형 사용하기

일반적으로 원시 타입 자료형을 사용하지만.. 아래 두 상황에선 기본 자료형을 wrap한 자료형이 사용됩니다.
1. nuallable 타입 연산(원시 타입은 null 불가능)
2. 타입을 제네릭으로 사용

숫자 연산의 경우 원시 타입과 wrap한 자료형의 성능 차이는 크지 않음

--------
객체 생성 비용 (메모리 이야기 좀 더 보기)
### 객체 헤더
- 64비트 JVM에서는 객체의 헤더가 12바이트를 차지합니다.

### 객체 헤더는 두 부분으로 나뉩니다:
- 8바이트는 마크 워드 (Mark Word)로 사용됩니다. 이 공간은 객체의 상태 (예: hash code, garbage collection information, locking information 등)를 저장합니다.
- 4바이트는 클래스 포인터로 사용됩니다. 이 포인터는 객체가 어떤 클래스의 인스턴스인지를 나타내며, 해당 클래스의 메타데이터를 가리킵니다.

## 객체의 정렬 (Object Alignment)
JVM은 객체를 8바이트 단위로 정렬합니다. 이는 메모리 접근을 최적화하고 성능을 향상시키기 위함
따라서 객체의 크기는 8바이트의 배수여야 합니다. 만약 객체의 실제 크기가 8바이트 배수가 아니라면, JVM은 객체 크기를 8바이트 단위로 맞추기 위해 패딩 바이트를 추가합니다.
따라서 헤더 12바이트 + 4바이트 = 16바이트가 최소 객체 크기

## 최소 객체 크기 (Minimum Object Size)
객체의 최소 크기는 16바이트입니다. 이는 객체 헤더(12바이트)와 최소 4바이트의 데이터 (또는 패딩)를 포함한 크기입니다.
예를 들어, 빈 객체의 경우에도 16바이트를 차지하게 됩니다.

그래서 계산을 어떻게 하는데?

```kt
class SimpleObject {
    int value;
}
```
- 객체 헤더: 12바이트
- int 필드: 4바이트
- 총 크기: 16바이트 (8의 배수로 정렬 필요 없음)

```kt
class ComplexObject {
    int value1;
    int value2;
}
```
- 객체 헤더: 12바이트
- int 필드 2개: 각 4바이트씩, 총 8바이트
- 총 크기: 20바이트 (8의 배수로 맞추기 위해 4바이트 패딩 추가)
- 실제 크기: 24바이트

알아두면 좋긴한데 계산하면서 까지 쓸일이 있을까는 싶습니다.

