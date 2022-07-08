# 아이템 36. 상속보다는 컴포지션을 사용하라

상속은 굉장히 강력한 기능이다.

`is-a`관계의 객체 계층 구조를 만들기 위해 설계되었다. 상속은 명확하지 않은 관계에 사용하면, 여러 문제가 발생할 수 있다.
따라서 단순 코드 추출 혹 재사용을 위해 상속을 하는 경우엔 일반적으로 컴포지션을 사용하는 것이 좋다.

## 간단한 행위 재사용
```kt
class ProfileLoader{
    fun load(){
        /*...*/
    }
}

class ImageLoader{
    fun load(){
        /*...*/
    }
}
```
보통 이런 경우 많은 개발자가 슈퍼 클래스를 만들어 사용합니다.
```kt
abstract class LoaderWithProgress{
    fun load(){
        innerLoad()
    }
    
    abstract fun innerLoad()
}

class ProfileLoader: LoaderWithProgress(){
    override fun innerLoad(){
        /*...*/
    }
}

class ImageLoader: LoaderWithProgress(){
    override fun innerLoad(){
        /*...*/
    }
}
```
아주 깔끔해 보이지만, 몇 가지 단점이 있다.
- 상속은 하나의 클래스만을 대상으로 가능하다. 행위를 추출하는 경우 거대해지는 BaseClass 가 생기고 굉장히 깊고 복잡한 계층 구조가 된다.
- 상속은 클래스의 모든 것을 가져온다. 불필요한 함수를 갖는 클래스가 생긴다(ISP 위반)
- 상속은 이해하기 어렵다.

이런 단점들 때문에 컴포지션을 사용한다.
`컴포지션: 객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용하는 것`
```kt
class Progress {
    fun showProgress() { /*show progress*/ }
    fun hideProgress() { /*hide progress*/ }
}

class ProfileLoader {
    val progress = Progress()
            
    fun load() {
        progress.showProgress()
        // read profile
        progress.hideProgress()
    }
}

class ImageLoader {
    val progress = Progress()
    
    fun load() {
        progress.showProgress()
        // read image
        progress.hideProgress()
    }
}
```
프로그레스 바를 관리하는 객체를 다른 모든 객체에서 갖고 활용하는 추가 코드가 필요하다.
이런 추가 코드를 처리하는 것이 조금 어려울 수 있어 컴포지션보다 상속을 선호하는 경우도 많다.
하지만 컴포지션의 사용으로 인해, 코드를 읽는 사람들이 코드의 실행을 더 명확하게 예측, 프로그레스 바를 훨씬 자유롭게 사용할 수 있다.

컴포지션의 활용으로 인해, 하나의 클래스 내부에서 여러 기능을 재사용할 수 있다.
상속을 활용해 하나 이상의 클래스를 상속하려 하면 두 기능을 하나의 슈퍼 클래스에 배치해야한다. 너무 복잡한 계층 구조가 만들어지니 안하는 것이 좋다.

## 모든 것을 가져올 수 밖에 없는 상속
상속은 슈퍼클래스의 메서드, 제약, 행위 등 모든 것을 가져와 계층 구조를 나타낼 때 굉장히 좋다.
하지만 일부분을 재사용하기 위한 목적으로는 적합하지 않다.

일부분만 재사용하려면 상속보단 컴포지션이 좋다.

컴포지션은 우리가 원하는 행위만 가져올 수 있기 때문이다.

## 캡슐화를 깨는 상속
상속을 활용할 때, 외부에서 활용하는 방법도 중요하지만 내부적으로 활용하는 방법도 중요하다.
내부적인 구현 방법 변경에 의해 캡슐화가 깨질 수 있기 때문이다.
```kt
class CounterSet<T>: HashSet<T>{
    var elementsAdd: Int = 0
        private set
    
    override fun add(element: T): Boolean {
        elementsAdd++
        return super.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdd += elements.size
        return super.addAll(elements)
    }
}
```
위 코드는 정상적이게 보이지만 동작하지 않습니다.

HashSet의 addAll 내부에서 add를 사용했기 때문입니다.

addAll과 add에서 추가한 요소 개수를 중복해서 세므로, 요소 3개를 추가해도 6이 출력된다.
addA 메소드를 지우면 문제가 사라지지만, 어느날 HashSet의 라이브러리가 업데이트 되어 addAll을 최적화하고 add를 내부적으로 호출하지 않으면 예상치 못한 형태로 동작할 것입니다.

이런 문제를 막으려면 컴포지션을 사용하면 된다.
```kt
class CounterSet<T> {
    private val innserSet = HashSet<T>()
    var elementsAdd: Int = 0
        private set
    
    fun add(element: T) {
        elementsAdd++
        innserSet.add(element)
    }
    
    fun addAll(elements: Collection<T>) {
        elementsAdd += elements.size
        innserSet.addAll(elements)
    }
}
```
위 처럼 코드를 수정할 경우 다형성이 사라진다. CounterSet은 더이상 Set이 아니게 된다.

이를 유지하고 싶으면 위임 패턴을 사용할 수 있다.

`위임패턴: 클래스가 인터페이스를 상속받게 하고, 포함한 객체의 메서드들을 활용해, 인터페이스에서 정의한 메서드를 구현하는 패턴`
`포워딩 메서드(forwarding method): 위임패턴으로 인터페이스에서 정의한 메서드가 구현된 method`
```kt
class CounterSet<T> : MutableSet<T> {
    private val innerSet: HahsSet<T>()
    var elementsAdd: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdd++
        innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdd += elements.size
        innerSet.addAll(elements)
    }
    
    override val size: Int
        get() = innerSet.size
        
    // 나머지 method...
}
```
이렇게 구현하면 포워딩 메서드가 너무 많아진다. 예제에는 9개나 구현됨

코틀린은 위임 패턴을 쉽게 구현할 수 있는 문법을 제공한다. -> 컴파일 시점에 포워딩 메서드가 생성됨

인터페이스 위임 활용 예시
```kt
class CounterSet<T>(
    private val innerSet: MutableSet<T> = mutableSetOf()
) : MutableSet<T> by innerSet {
    var elementsAdd: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdd++
        innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdd += elements.size
        innerSet.addAll(elements)
    }
    
    // 나머지 method...
}
```

## 오버라이딩 제한하기
상속용으로 설계되지 않은 클래스를 상속하지 못하게 하려면, `final`을 사용하면 된다.
이런 경우 `open` 키워드를 사용한다.
```kt
open class Parent{
    fun a() {}
    open fun b() {}
}

class Child: Parent() {
    override fun a() {} // error
    override fun b() {}
}
```
상속용으로 설계된 메서드에만 open을 붙이면 된다. 메서드 오버라이드를 할 때, 서브클래스에서 해당 메서드에 final을 붙일 수 있다. -> 오버라이드 메서드 제한

### 정리
- 컴포지션은 더 안전하다. 다른 클래스의 내부적인 구현에 의존x, 외부에서 관찰되는 동작에만 의존
- 더 유연하다. 상속은 한 클래스만을 대상으로 하고 모든 것을 받는다, 컴포지션은 여러 클래스를 대상으로 가능하고 필요한 것만 받을 수 있다.
- 더 명시적이다. 리시버를 명시적으로 활용해야한다.
- 생각보다 번거롭다. 객체를 명시적으로 사용해야 하므로, 대상 클래스에 일부 기능을 추가할 때 포함하는 객체의 코드를 변경해야한다.
- 상속은 다형성을 활용할 수 있다. 양날의 검이므로 슈퍼클래스와 서브클래스의 규약을 잘 지켜 코드를 작성해야한다.
