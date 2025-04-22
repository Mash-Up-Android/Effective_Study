# Unit?을 리턴하지 마라

## Unit 이란?

```kotlin
fun sayHello() {
    println("Hello")
}

```

이 함수는 아무것도 `return`  하지 않는데

그렇게 되면 코틀린은 **자동으로 `Unit`을 리턴한다**

> Unit = "아무것도 안 돌려줌"
마치 자바의 void와 같다
>

```kotlin
fun sayHello(): Unit {
    println("Hello")
}
```

이렇게 써도 되지만, 코틀린에서는 `: Unit`은 주로 생략한다

`Unit?`은 `null`이 될 수도 있는 Unit

| 타입 | 의미 |
| --- | --- |
| `Unit` | 값 없음 (void 와 같음) |
| `Unit?` | 값이 없는데 아예 `null`일 수도 있음 |

즉, **"아무 값도 없을 수도 있고, 있을 수도 있음"** → `Unit?`

벌써 애매하고 읽는데 생각을 한 번 더 해야한다

하지만 이처럼

Unit? 타입은 Unit 또는 null 값을 가질 수 있어서,

Boolean의 true / false 처럼 사용할 수 있다.

```kotlin
fun keyIsCorrect(key: String) : Boolean
if(!keyIsCorrect(key)) return

fun verifyKey(key: String) : Unit?
verifyKey(key) ?: return
```

두 코드는 동일한 로직인데

Unit? 으로 Boolean을 표현한다는 것 자체가 가독성이 안좋다.

예측하기 어렵고 이해하기 어렵다.

다음 코드처럼 쓸 수 있지만 if-else 조건문이 훨씬 가독성이 좋다

```kotlin
getData()?.let { view.showData(it) } ?: view.showError()

if (person != null && person.isAdult) {
		view.showPerson(person)
} else {
		view.showError()
}
```

최대한 Boolean을 활용하고, Unit?을 리턴하지 말자