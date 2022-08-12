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
