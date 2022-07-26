# 아이템 39. 태그 클래스보다는 클래스 계층을 사용하라

상수 '모드'를 가진 클래스를 태그라고 부르며, 태그를 포함한 클래스를 태그 클래스라고 부릅니다.

하지만 태그 클래스는 다양한 문제를 내포합니다.
서로 다른 책임을 한 클래스에 태그로 구분해서 넣는다는 것에서 시작합니다.
```kt
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
) {
    fun match(value: T?) = when(matcher) {
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
    }
    
    enum class Matcher {
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }
    
    companion object {
        fun <T> equal(value: T) =
            ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)
            
        fun <T> notEqual(value: T) =
            ValueMatcher<T>(value = value, matcher = Matcher.NOT_EQUAL)
            
        fun <T> emptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)
            
        fun <T> notEmptyList() =
            ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
    }
}
```
이런 접근 방법에는 많은 단점이 있습니다.
- 한 클래스에 여러 모드를 처리하기 위한 상용구(bolierplate) 추가됨
- 여러 목적으로 사용해야해 프로퍼티가 일관적이지 않게 사용될 수 있고, 더 많은 프로퍼티 필요
- 요소가 여러 목적을 가지고, 요소를 여러 방법으로 설정할 수 있는 경우에는 상태의 일관성과 정확성을 지키기 어려움
- 팩토리 메서드를 사용해야 하는 경우가 많음. 아닌 경우 객체가 제대로 생성되었는지 확인이 어려움

So, 코틀린에서는 일반적으로 태그클래스보다 sealed 클래스를 많이 사용함
```kt
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean
    
    class Equal<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value == this.value
    }
    
    class NotEqual<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value != this.value
    }
    
    class EmptyList<T>(): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value is List<*> && value.isEmpty()
    }
    
    class NotEmptyList<T>(): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value is List<*> && value.isNotEmpty()
    }
}
```
책임이 분산되어 훨씬 깔끔해짐

## sealed 한정자

sealed 한정자는 외부 파일에서 서브 클래스를 만드는 행위를 모두 제한 -> 

외부에서 추가적인 서브 클래스를 만들수 없으므로 타입이 추가되지 않음이 보장됨 ->

when을 사용할 때 else를 안써도 됨

```kt
fun <T> ValueMatcher<T>.reversed(): ValueMatcher<T> =
when (this) {
    is ValueMatcher.EmptyList -> ...
    is ValueMatcher.NotEmptyList -> ...
    is ValueMatcher.Equal -> ...
    is ValueMatcher.NotEqual -> ...
}
```

만약 sealed를 사용하지 않고 abstract 키워드를 사용하면, 다른 개발자가 새로운 인스턴스를 만들어 사용할 수 있음 ->

함수를 abstract로 선언하고, 서브클래스 내부에 구현해야함

sealed를 사용하면, 확장 함수를 사용해 클래스에 새로운 함수를 추가하거나, 클래스의 다양한 변경 쉽게 처리 가능

> 클래스의 서브 클래스를 제어하려면 sealed 한정자 / abstract는 상속과 관련된 설계

## 태그 클래스와 상태 패턴를 혼동하지 마라

> 상태 패턴 : 객체의 내부 상태가 변화할 때, 객체의 동작이 변하는 소프트웨어 디자인 패턴, 컨트롤러, 프레젠터, 뷰 모델을 설계할 때 많이 사용됨(MVC, MVP, MVVM..)

## 정리
- 코틀린에서는 태그 클래스보다 타입 계층을 사용하는 것이 좋음(sealed class)
- sealed class는 상태 패턴과 다름
- 타입 계층과 상태 패턴은 함께 사용하는 협력 관계
