# 아이템 42. compareTo의 규약을 지켜라

`compareTo`는 수학적인 부등식으로 변환되는 연산자이다. 같으면 0 리턴 / 크면 양수 리턴 / 작으면 음수를 리턴한다.
이 함수는 `Comparable<T>` 인터페이스에 있으며 어떤 객체가 이 인터페이스를 구현하고 있거나 **`compareTo`라는 연산자 메서드를 가진 의미는 객체가 순서를 가지고 있다고 봐야합니다.**

`compareTo`의 동작
- 비대칭적 동작 : a>=b 이고 b>=a면, a==b이다. 비교와 동등성 비교에 어떤 관계가 있어야 하고 서로 일관성이 있어야 한다.
- 연속적 동작 : a>=b이고 b >= c면, a>=c이다. 마찬가지로 a>b이고b>c면, a>c이다. 이런 동작을 못하면, 요소 정려이 무한 반복에 빠질 수 있다.
- 코넥스적 동작 : 두 요소는 어떤 확실한 관계를 가져야 한다. 즉, a>=b 혹 b>=a 중 적어도 하나는 항상 true 이다.
두 요소 사이에 관계가 없으면, 퀵 정렬과 삽입 정렬 등의 고전적인 정렬 알고리즘을 사용할 수 없다. 대신 위상 정렬 알고리즘은 사용 가능하다.

# compareTo를 따로 정의해야 할까?
코틀린에서 `compareTo`를 따로 정의할 상황은 거의 없다.일반적으로 프로퍼티 하나로 순서를 지정하는 것이 충분하기 때문이다.

`sortedBy`를 통해 원하는 프로퍼티로 컬렉션 정렬을 할 수 있다.
```kt
class User(val name: String, val surname: String)
val names = listOf<User>(/*...*/)

val sorted = names.sortedBy { it.surname }
```

여러 프로퍼티를 기반으로 정렬할 경우 `sortedWith` 를 사용하면 된다.
```kt
val sorted = names
    .sortedWith(compareBy({ it.surname }, { it.name }))
```
위 코드는 `surname`이 같은 경우 `name`까지 비교해 정렬한다.

대한 절대적인 기준이 없다면, 아예 비교하지 못하게 만드는 것도 좋다.

문자열은 알파벳과 숫자 등의 순서가 있어 내부적으로 `Comparable<String>`을 구현하고 있습니다. 이는 굉장히 유용하지만 단점이 있다.

부등호 기호를 사용해 문자열을 비교하면 가독성이 떨어진다.
```kt
// 가독성 떨어짐
println("Kotlin" > "Java") // true
```
측정 단위, 날짜, 시간과 같이 자연스러운 순서를 갖는 객체들과 달리 순서가 자연스러운지 확실하지 않은 객체가 있다면, `comparator`를 사용하는 것이 좋다.
이를 자주 사용하면 클래스에 `companion object`로 만들어 두는 것도 좋다.
```kt
class User(val name: String, val surname: String){
    // ...
    
    companion object {
        val DISPLAY_ORDER = compareBy(User::surname, User::name)
    }
    
    val sorted = names.sortedWith(User.DISPLAY_ORDER)
}
```

# compareTo 구현하기
`compareTo`를 구현할 때 유용하게 활용할 수 있는 톱레벨 함수가 있다.
두 값을 단순하게 비교하면, `compareValues` 함수를 다음과 같이 활용할 수 있다.
```kt
class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
            compareValues(surname, other.surname)
}
```

더 많은 값을 비교하거나,선택기를 활용해 비교하고 싶다면, `compareValuesBy`를 사용한다.
```kt
class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
            compareValuesBy(this, other, { it.surname }, { it.name })
}
```
이 함수는 비교기를 만들 때 도움이 된다. 이 함수는 `compareTo`와 같이 아래 값을 리턴해야 한다.
1. 0 : 리시버와 other가 같은 경우
2. 양수 : 리시버가 other보다 큰 경우
3. 음수 : 리시버가 other보다 작은 경우

이후 함수가 비대칭 동작, 연속적 동작, 코넥스적 동작을 하는지 확인해야한다.
