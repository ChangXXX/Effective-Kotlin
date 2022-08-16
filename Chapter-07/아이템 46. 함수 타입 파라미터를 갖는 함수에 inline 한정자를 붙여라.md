# 아이템 46. 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

코틀린 stdlib의 고차함수는 대부분 inline 한정자가 붙어 있다.

왜 inline 한정자를 사용할까?
1. 타입 아규먼트에 reified 한정자를 붙여 사용 가능
2. 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
3. 비지역적 리턴 사용 가능

## 타입 아규먼트에 reified 한정자를 붙여 사용 가능
구버전 JAVA는 제네릭이 없었고 나중에 사용가능하게 되었지만.. JVM 바이트 코드에는 제네릭이 없다.

따라서 컴파일시 제네릭 타입과 관련 내용이 제거된다.

`List<Int>`는 컴파일시 `List`로 변경된다.
```kt
// error
fun <T> printTypeName() {
    print(T::class.simpleName)
}

// use
inline fun <reified T> printTypeName(){
    print(T::class.simpleName)
}

printTypeName<Int>() // 코드
print(Int::class.simpleName) // 컴파일 시 변경되는 코드
```

함수를 inline으로 만들어 사용하면 위 코드와 같이 제네릭 내용이 사라지지 않는 것을 확인 할 수 있다.

reified 한정자를 지정해 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체된다.

또한, refied를 사용하면 특정 타입을 확인할 때 사용할 수 있다.

## 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
모든 함수는 inline을 붙이면 조금 더 빠르게 동작한다.

함수 호출과 리턴을 위해 점프하는 과정과 백스택을 추적하는 과정이 없기 때문이다.

이런 이유로 stdlib의 간단한 함수들은 inline을 사용한다.

이런 장점이 있지만, 함수 파라미터를 가지지 않는 함수가 inline을 사용해도 큰 차이가 발생하지 않습니다.
그 이유는 ?
1. 함수를 객체로 조작할 때 발생하는 문제 이해해야 함
2. 코틀린/JS에서는 자바스크립트가 함수를 일급 객체로 처리해 굉장히 간단하게 변환
3. 코틀린/JVM에서는 JVM 익명 클래스 또는 일반 클래스를 기반으로, 함수를 객체로 만들어 냄

```kt
val lambda: () -> Unit = {
    // code..
}
```
위 코드는 결국 클래스로 컴파일되고, 익명 클래스로 컴파일되면 아래와 같다.
```kt
Function0<Unit> lambda = new Function0<Unit>() {
    public Unit invoke() {
        // code..
    }
}
```
혹 별도의 파일에 정의된 일반 클래스로 컴파일하면 아래와 같다.
```kt
public class Test$lambda implements Function0<Unit> {
    public Unit invoke() {
        // code..
    }
}

// use
Function0 lambda = new Test$lambda()
```
익명 클래스, 일반 클래스로 컴파일 되는 것의 큰 차이는 없다.

함수 본문을 객체로 wrap하게되면 코드의 속도는 느려진다.
```kt
// faster
inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}

// slower
fun repeatNoInline(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}
```
위 인라인 함수의 코드는 평균 189ms로 동작하고, 일반 함수는 평균 477ms로 동작
why?
첫 번째 함수는 숫자로 반복을 돌고, 빈 함수를 호출
두 번재 함수는 숫자로 반복을 돌며, 객체를 호출하고, 이 객체가 빈 함수를 호출

인라인과 인라인이 아닌 함수의 가장 중요한 차이는 함수 리터럴 내부에서 지역 변수를 캡처할 때 확인할 수 있다.

캡처된 값은 객체로 wrapping 해야하며, 사용할 때 마다, 객체를 통한 작업이 이뤄짐

만일 아래와 같은 코드를 구현하였다면..
```kt
var l = 1L
noinlineRepeat(100_000_000) {
    l += it
}
```
컴파일 시 아래와 같아집니다.
```kt
val a = Ref.LongRef()
a.element = 1L
noinlineRepeat(100_000_000){
    a.element = a.element + it
}
```
즉, 인라인이 아닌 람다 표현식에서는 지역 변수 l을 직접 사용할 수 없습니다.
l은 컴파일 과정 중 래퍼런스 객체로 래핑되고, 람다 표현식 내부에서는 이를 사용합니다.

## 비지역적 리턴 사용 가능

```kt
// error
fun repeatNoInline(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}

fun main() {
    repeatNoInline(10) {
        print(it)
        return 
    }
}

// OK
fun main() {
    repeat(10) {
        print(it)
        return // OK
    }
}
```
위 코드를 살펴봅시다.
위 코드가 안되는 이유는 함수 리터럴이 컴파일될 때, 함수가 객체로 래핑되어 발생하는 문제

함수가 다른 클래스에 위치하므로, return을 사용해 main으로 돌아올 수 없음

## inline 한정자의 비용
inline 한정자는 유용하지만 모든 곳에서 사용할 수 없음
- 재귀적 동작 불가능
- 많은 가시성 제한을 가진 요소 사용 불가
  - public inline 함수 내부에서 private과 internal 가시성을 가진 함수/프로퍼티 사용 불가
- 구현을 숨길 수 없어 클래스에 거의 사용되지 않음
- 코드의 크기가 쉽게 커짐

## crossinline과 noinline
함수를 인라인으로 만들고 싶지만, 어떤 이유로 일부 함수 타입 파라미터는 inline으로 받고 싶지 않은 경우가 있을 수 있음

이런 경우 아래와 같은 한정자 사용
- `crossinline` : 아규먼트로 인라인 함수를 받지만, 비지역적 리턴을 하는 함수는 받을 수 없게 만듬, 인라인이 아닌 람다 표현식과 조합해 사용할 때 문제가 발생하는 경우 활용
- `noinline` : 아규먼트로 인라인 함수를 받을 수 없게 만듬, 함수를 아규먼트로 사용하고 싶을 때 활용
