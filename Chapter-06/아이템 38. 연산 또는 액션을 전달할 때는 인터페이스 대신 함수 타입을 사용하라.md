# 아이템 38. 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라

대부분 프로그래밍 언어는 함수 타입이라는 개념이 없다. 그래서 연산 또는 액션을 전달할 때 메서드가 하나만 있는 인터페이스를 활용한다.

이러한 인터페이스를 `SAM(Single-Abstract Method)`라고 한다.

```kt
interface OnClick {
    fun clicked(view: View)
}
```

함수가 SAM을 받는다면, 이런 인터페이스를 구현한 객체를 전달받는다는 의미이다.
```kt
fun setOnclickListener(listener: Onclick) {
    // ...
}

setOnClickListener(object: Onclick {
    override fun clicked(view: View) {
        // ...
    }
})
```

이런 코드를 함수 타입이 사용하는 코드로 변경하면, 더 많은 자유를 얻을 수 있다.
```kt
fun setOnClickListener(listener: (View) -> Unit) {
    // ...
}
```
다음과 같은 방법으로 파라미터를 전달할 수 있습니다
- 람다 표현식 혹 익명 함수
```kt
setOnClickListener { /*...*/ }
setOnClickListener(fun(view) { /*...*/ })
```
- 함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달
```kt
setOnClickListener(::println)
setOnClickListener(this::showUsers)
```
- 선언된 함수 타입을 구현한 객체로 전달
```kt
class ClickListener: (View)->Unit {
	override fun invoke(view: View) {
		// ...
	}
}
```
사람들은 SAM의 장점이 아규먼트에 이름이 붙어 있는 것이라고 합니다. 하지만 타입 별칭을 사용하면, 함수 타입도 이름을 붙일 수 있습니다.
SAM만의 장점이 아니라는 뜻입니다.
```kt
typealias Onclick = (View) -> Unit
```

## 언제 SAM을 사용해야 할까?

딱 한가지 경우에 SAM을 사용하는 것이 좋다.
코틀린이 아닌 다른 언어에서 사용할 클래스를 설계할 때이다.
