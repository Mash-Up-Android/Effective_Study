# 22. 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

---

---

# 개요

제네릭 함수 : 함수에 타입을 전달할 수 있는 타입 아규먼트를 사용하는 함수

타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여, 컴파일러가 타입을 더 정확하게 추측하게 해줌

⇒ 프로그램이 더 안전해지고, 개발자는 편리함

예시 : stdlib의 filter 함수가 대표적

```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> { 
    val destination = ArrayList<T>()
    for (element in this) {
        if (predicate(element)) { 
            destination.add(element) 
        } 
    }
    return destination
}
```

위 예시의 `filter` 함수의 람다 표현식 내부에서, 컴파일러는 아규먼트가 컬렉션의 요소와 같은 타입이라는 것을 알 수 있으므로, 잘못 처리하는 것을 막을 수 있음

제네릭은 기본적으로 `List<String>`, `Set<User>`처럼 구체적인 타입으로 컬렉션을 만들 수 있게 클래스와 인터페이스에 도입된 기능

→ 컴파일 과정에서 최종적으로 이런 타입 정보는 사라지지만, 개발 중에는 특정 타입을 사용하게 강제 가능

이 같은 기능은 정적 타입 프로그래밍 언어에서는 굉장히 유용함

---

# 제네릭 제한

타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입만 사용하게 타입을 제한하는 것임

다음 코드는 콜론 뒤에 슈퍼타입을 설정해서 제한을 걸음

```kotlin
fun <T: Comparable<T>> Iterable<T>.sorted(): List<T> {
    /* ... */
}

fun <T, C : MutableCollection<in T>> Iterable<T>.toCollection(destination: C): C {
    /* ... */
}

class ListAdaptor<T: ItemAdapter>(/*...*/) { /*...*/ }
```

타입에 제한이 걸리므로, 내부에서 해당 타입이 제공하는 메서드를 사용 가능

예를 들어 `T`를 `Iterable<Int>`의 서브타입으로 제한하면, `T`타입을 기반으로 반복 처리가 가능하고, 반복 처리 때 사용되는 객체가 `Int`라는 것을 알 수 있음

또한 `Comparable<T>`로 제한하면, 해당 타입을 비교할 수 있다는 것을 알 수 있음

많이 사용하는 제한으로는 `Any`가 있는데, 이는 nullable이 아닌 타입을 나타냄

```kotlin
inline fun <T, R : Any> Iterable<T>.mapToNull(
    transform: (T) -> R?
): List<R> {
    return mapNotNullTo(ArrayList<R>(), transform)
}
```

둘 이상의 제한도 가능

```kotlin
fun <T: Animal> pet(animal: T) where T: GoodTempered {
    /*...*/
}

fun <T> per(animal: T) where T: Animal, T: GoodTempered {
    /*...*/
}
```

---

# 정리

- 코틀린 자료형 시스템에서 타입 파라미터는 굉장히 중요
- 일반적으로 이를 사용해서 type-safe 제네릭 알고리즘과 제네릭 객체를 구현함
- 타입 파라미터는 구체 자료형(concrete type)의 서브타입을 제한 가능
- 이렇게 하면 특정 자료형이 제공하는 메서드를 안전하게 사용 가능

---