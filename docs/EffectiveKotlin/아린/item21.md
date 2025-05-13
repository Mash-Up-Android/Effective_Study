# 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

## **프로퍼티 위임**을 활용하여 **코드의 중복을 줄이고, 재사용성을 높이자**

### **프로퍼티 위임이란?**

**속성값을 직접 관리하지 않고** 별도의 객체에게 **위임**하는 방식
즉, 속성 값을 어떻게 처리할지에 대한 로직을 **위임자**에게 맡김 ⇒ Kotlin의 `by` 키워드

- **중복 코드 줄이기**: 같은 로직을 여러 곳에서 반복하지 않고, 하나의 공통된 위임 객체에 맡김
- **재사용성**: 프로퍼티 위임을 사용하면 같은 패턴을 여러 클래스에서 손쉽게 재사용할 수 있음
- **가독성 향상**: 코드를 간결하고 이해하기 쉽게 만들어 줌

```kotlin
class User {
    var name: String by Delegates.observable("이름없음") { _, oldValue, newValue ->
        println("이름이 $oldValue 에서 $newValue 으로 변경되었습니다")
    }
}

fun main() {
    val user = User()
    user.name = "아딘"  // 이름이 이름없음 에서 아딘 으로 변경되었습니다
    user.name = "아린"    // 이름이 아딘 에서 아린 으로 변경되었습니다
}
```

위의 예시에서 `name` 프로퍼티는 **`observable`** 위임을 사용했음

`observable`은 프로퍼티 값이 변경될 때마다 콜백을 호출하여 변경 사항을 처리함

**여기서 프로퍼티 값이 변경될 때마다 로그를 찍어주는 로직을 한 곳에만 작성하여 재사용성을 높임**

```kotlin
private val viewModel: LoginViewModel by viewModel()
```

위 코드는 익숙하다

의존성 주입, 리소스 / 데이터 바인딩에서도 많이 볼 수 있었다

## lazy

`lazy` 위임은 프로퍼티 값을 **첫 번째 접근 시에만 계산**하도록 할 때 사용

```kotlin
val lazyValue: String by lazy {
    println("먀!")
    "나는 아딘이야"
}

fun main() {
    println(lazyValue)  // 먀! 나는 아딘이야
    println(lazyValue)  // 나는 아딘이야 
}

```

두번째에서 먀! 출력 안됨

## vetoable

`vetoable` 위임은 **값을 변경하기 전에 조건을 검사**하고 조건에 맞으면 변경, 그렇지 않으면 변경을 거부함

```kotlin
var vetoableValue: Int by Delegates.vetoable(0) { _, oldValue, newValue ->
    newValue >= 0  // 음수로는 변경되지 않도록
}

fun main() {
    vetoableValue = 5   // 값이 5로 설정됨
    vetoableValue = -3  // 값 변경되지 않음, 5로 유지
}
```

## notNull

처음에는 초기화되어 있지 않지만 null은 들어갈 수 없고 반드시 나중에 값을 할당해야 함

값을 사용하려고 하는데 초기화 안 되어 있으면 예외가 발생

```kotlin
class Person {
    var name: String by Delegates.notNull()
}

fun main() {
    val person = Person()
    // println(person.name) => 초기화 전에 접근하면 예외
    person.name = "아딘"  
    println(person.name) // 아딘
}

```

음 ..이번 아이템 너무 어려웠다 ㅜㅜ