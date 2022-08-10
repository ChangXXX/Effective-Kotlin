# 아이템 44. 멤버 확장 함수의 사용을 피하라

- 클래스에 대한 확장함수를 정의할 때, 이를 멤버로 추가하는 것은 좋지 않다.
- 확장함수는 첫번째 아규먼트로 리시버를 받는 단순한 일반 함수로 컴파일된다.
```kt
// 컴파일 전
fun String.isPhoneNumber(): Boolean = 
	  length == 7 && all { it.isDigit() } 

// 컴파일 후
fun isPhoneNumber(`$this`: String): Boolean = 
	  '$this'.length == 7 && '$this'.all { it.isDigit() }
```
위와 같이 확장함수는 컴파일 시점에 단순하게 변환된다.

확장 함수를 클래스 멤버 함수로 정의할 수도 있고, 인터페이스 내부에 정의할 수 있다.

하지만 가시성의 제한을 위해 확장 함수를 멤버로 정의하는 것은 좋지 않다.

## 확장 함수를 멤버 함수로 정의하지 않는 이유

1. 가시성을 제한하지 못한다. -> 단순히 사용이 어려워짐
```kt
PhoneBookIncorrect().apply { "01000000000".test() }
```
가시성을 제한하고 싶다면 private을 붙이면 된다. 
```kt
private fun String.isPhoneNumber() =
    ...
```

2. 레퍼런스를 지원하지 않는다.
```
val ref = String::isPhoneNumber
val str = "1234567890"
val boundedRef = str::isPhoneNumber

val refX = PhoneBookIncorrect::isPhoneNumber // 오류
val book = PhoneBookIncorrect() 
val bookRefX = book::isPhoneNumber // 오류
```
3. 암묵적 접근을 할 때, 두 리시버 중 어떤 리시버가 선택될지 혼동
```kt
class A {
    val a = 10
}

class B {
    val a = 20
    val b = 30
    
    fun A.test() = a + b // 답은 40? 50?
}

fun main() {
    B().apply { println(A().test()) } // 40
}
```
4. 확장 함수가 외부에 있는 다른 클래스를 리시버로 받을 때, 해당 함수가 어떤 동작을 하는지 명확하지 않음

```kt
class A {
    var a = 0
}
class B {
    var a = 5
    var b = 10
    
    fun A.update() = a + 100 // A와 B 중 어느 것 업데이트?
}

fun main() {
    B().apply { println(A().update()) } // 100
}
```
5. 경험이 적은 개발자의 경우 확장 함수를 보면, 겁먹음

멤버 확장 함수를 사용하는 것이 의미가 있다면 사용해도 괜찮다. 하지만, 단점을 인지하고 사용하지 않는 편이 더 좋다.
