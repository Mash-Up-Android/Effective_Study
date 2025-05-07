# 18. 코딩 컨벤션을 지켜라

---

---

# 개요

코틀린 컨벤션 공식문서 : [https://kotlinlang.org/docs/coding-conventions.html](https://kotlinlang.org/docs/coding-conventions.html)

코틀린은 굉장히 잘 정리된 코딩 컨벤션을 갖고 있음

이런 컨벤션이 모든 프로젝트에 최적은 아니지만, 코틀린 컨벤션을 최대한 지켜주는 게 좋음

코딩 컨벤션의 장점

- 어떤 프로젝트를 접해도 쉽게 이해 가능
- 다른 외부 개발자도 프로젝트 코드를 쉽게 이해 가능
- 다른 개발자도 코드의 작동 방식을 쉽게 추측 가능
- 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉬움

컨벤션에 도움 되는 도구

- Intellij 포매터 : 공식 코딩 컨벤션 스타일에 맞춰서 코드를 변경해 줌
- ktlink : 많이 사용되는 코드를 분석하고 컨벤션 위반을 알려주는 linter

코틀린은 많은 코틀린 개발자가 이전엔 자바 개발자였기 때문에 자바의 코딩 컨벤션을 잘 따름

자주 위반되는 규칙중 하나는 많은 파라미터를 갖고 있는 클래스나 함수는 파라미터를 한줄씩 작성

```kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = ""
) : Human(id, name) {
    // ...
}
```

```kotlin
public fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = "",
    limit: Int = -1,
    truncated: CharSequence = "...",
    transform: ((T) -> CharSequence)? = null
) : String {
	// ...
}
```

다음과 같은 코드는 하면 안됨

```kotlin
class Person(val id: Int = 0,
    val name: String = "",
    val surname: String = "") : Human(id, name) { 
    // ...
}
```

- 모든 클래스 아규먼트가 클래스 이름에 따라 다른 크기의 들여쓰기를 가짐. 이런 형태로 작성하면, 클래스 이름을 변경할 때 모든 기본 생성자 파라미터의 들여쓰기를 조정해야 함
- 클래스가 차지하는 공간의 너비가 너무 큼. 처음 class 키워드가 있는 줄도 너비가 너무 크고, 이름이 가장 긴 마지막 파라미터와 슈퍼클래스 지정이 함께 있는 줄도 너무 큼

프로젝트 컨벤션은 반드시 지켜야 함 → 여러사람이 작성해도 한사람이 작성한 것처럼!

---

# 내 생각

그냥 코딩 컨벤션은 무죅건 지키자

---