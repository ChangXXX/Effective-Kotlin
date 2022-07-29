# 아이템 40. equals의 규약을 지켜라

## 동등성
- 구조적 동등성: `equals`메서드와 이를 기반으로 만들어진 ==연산자(!= 포함)로 확인하는 동등성
- 레퍼런스 동등성: ===연산자(!== 포함)로 확인하는 동등성

연산자를 다른 타입의 두 객체를 비교하면 안되고 같은 타입을 비교하거나 상속 관계의 경우 비교 가능
```kt
open class Animal
class Book
Animal() == Book() // X
Animal() === Book() // X

class Cat: Animal()
Animal() == Cat()
Animal() === Cat()
```

## equals가 필요한 이유
두 객체의 프로퍼티가 같다면 같은 객체로 볼 수 있습니다.
```kt
data class Name(val name: String, val surName: String)
val name1 = Name("Marcin", "Moskala")
val name2 = Name("Marcin", "Moskala")
val name3 = Name("Maja", "Moskala")

name1 == name1 // true
name1 == name2 // true 데이터가 같아서
name1 == name3 // false

name1 === name1 // true
name1 === name2 // false
name1 === name3 // false
```
위 코드 예시에서 확인하는 바와 같이 `data` 한정자를 사용해 data class를 정의하면 동등성이 위와 같이 동작합니다.

데이터 클래스는 위처럼 모든 프로퍼티를 비교하지 않는 경우가 아니라 일부 프로퍼티만 비교할 때도 유용하다.

### equals를 직접 구현하는 경우
- data class를 사용해 제공되는 동작과 다른 동작을 할 필요가 있을 때
- 일부 프로퍼티만으로 비교해야 하는 경우
- data 한정자를 붙이는 것을 원하지 않거나 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

## equals의 규약
코틀린 1.4.31 기준 주석
- 반사적동작 : null이 아닌 값이면 true 리턴
```kt
// 이렇게 하면 안된다
class Time (
	val millisArg: Long = -1,
	val isNow: Boolean = false,
) {
	val milllis: Long get () =
		if (isNow) System.currentTimeMillis()
		else millis Arg

	override fun equals(other: Any?): Boolean = 
		other is Time && millis == other.millis
}

val now = Time(isNow = true)
now == now // 때로는 true, 때로는 false
List(100000) { now }.all { it == now }
// 대부분 false
```
위와 같이 equals 규약이 잘못되면, 내부에 해당 객체가 포함되어 있어도 contains 메서드 등으로  포함됐는지 확인할 수 없습니다.

- 대칭적 동작 : x와 y가 널이 아니면 x.equals(y)는 y.equals(x)와 같은 결과 출력
```kt
class Complex(
    val real: Double,
    val imaginary: Double
) {
// 이렇게 하면 안됨. 비대칭적
	  override fun equals() {
		    if (other is double) {
            return imaginary == 0.0 && real == other
		    }
		    return other is Complex && 
			      real == other.real &&
				    imaginary == other.imaginary
	}
}

Complex(1.0, 0.0).equals(1.0) // true
1.0.equals(Complex(1.0, 0.0)) // false
```
대칭적 동작을 하지 못하는 것은 contains 함수와 단위 테스등에서 예측하지 못한 동작 발생 가능
```kt
val list = listOf<Any>Complex(1.0, 0.0))
list.contains(1.0) // false 컬렉션 구현에 따라 결과가 달라질 수 있음
```
- 연속적 동작 : x, y, z가 널이 아닌 값이고 x.equals(y)와 y.equals(z)가 true라면 x.equals(z)도 true이다.
```kt
val o1 = DateTime(Date(1992, 10, 20), 12, 30 ,0)
val o2 = DateTime(Date(1992, 10, 20))
val o3 = DateTime(Date(1992, 10, 20), 14, 45, 30)

o1 == o2 // true
o2 == o3 // true
o1 == o3 // false
```
- 일관적 동작 : x와 y가 널이 아닌 값이면, x.equals(y)는 여러번 실행해도 항상 같은 값을 반환
- 널과 관련된 동작 : x가 널이 아닌 값이라면, x.equals(null)은 항상 null을 반환

### URL과 관련된 equals 문제
- 네트워크 상태에 따라 결과가 달라질 수 있음
- 일반적으로 equals와 hashCode 처리는 빠를 거라 예상하지만, 네트워크 처리는 굉장히 느림 -> URL을 set에 추가하는 기본적 조작도 쓰레드를 나눠해야함
- 동일한 IP 주소를 갖는다고, 동일한 콘텐츠를 나타내지 않는다.

### equals 구현하기
- 특별한 이유가 아닌 이상 직접 구현 하는 것은 좋지 않음
- 구현하는 경우 반사적, 대칭적, 연속적, 일관적 동작을 꼭 확인해야함
- final class로 만드는 것이 좋음
- 상속을 하는 경우 서브클래스에서 equals 동작을 수정하면 안됨
