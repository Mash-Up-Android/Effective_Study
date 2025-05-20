# **일반적인 알고리즘을 구현할 때 제네릭을 사용하라**

## 제네릭 함수

- 타입 아규먼트를 사용하는 함수

```kotlin
fun printInt(value: Int) {
		println(value)
}
```

일반 함수는 위처럼 타입이 고정되어 있고, Int 외의 String 등 다른 타입을 쓸 수 없다

```kotlin
fun <T> printValue(value: T) {
		println(value)	
}
```

<T>가 타입 아규먼트임.

- T가 Int면 Int로, String이면 String으로 작동

> **일반적인 로직을 여러 타입에서 반복해서 만들지 말고 제네릭(Generic)을 사용해서 재사용성과 타입 안정성을 높이자**
>

ilter, map, reduce 등…

타입 파라미터는 타입에 관련된 정보를 컴파일러에 제공함으로써,
컴파일러가 조금이라도 타입을 더 정확하게 추측하도록 돕는다

이로써, 프로그램이 더 안전해지고 개발하기 편리해 진다

filter 함수에서의 람다 표현식 내부를 예시로 보자

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
}
```

아규먼트가 컬렉션 요소와 같은 타입임을 컴파일러가 알 수 있으므로, 예외 처리가 용이하다
또한 이를 기반으로 IDE 에서 여러가지 유용한 제안을 해주어서 프로그래밍이 편리하다

<T>

- 타입 파라미터

predicate: (T) -> Boolean

- "T 타입을 받아서 true/false를 반환하는 조건 함수”

```kotlin
val users: Set<User> = setOf(User("곰"), User("토끼"))

val user = users.first()
println(user.name) // 타입 따로 검사 안 해도 User로 바로 사용 가능
```

이 Set은 User 객체만 담음
Set에서 꺼내면, 컴파일러는 User 타입임을 알고 있음

📌 컴파일 과정에서 타입 정보는 사라진다?

→ 코틀린은 JVM 위에서 돌아가는데, JVM은 실행할 땐 타입 정보를 안 씀
→ 즉, <> 안의 내용은 개발할 때 유효하고, 실행할 땐 빠짐

하지만 우리가 개발하는 동안엔 그 정보 덕분에 타입을 강제할 수 있고
타입 실수도 막고, 자동 완성도 잘 되며 코드가 훨씬 안전해짐

## 제네릭 제한

### sorted()

```kotlin
<T : 어떤타입>
```

T는 반드시 “어떤타입”의 하위 타입이어야 한다
어떤타입을 상속 or 구현했어야 함

```kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T>

```

T는 반드시 Comparable 인터페이스를 구현한 타입만 들어올 수 있음
→ 정렬하려면 크기 비교가 가능해야 하잖아? (<, > 같은 거)

이 조건이 없으면 컴파일러는 T의 크기 비교가 가능한지 아닌지 알 수 없음

예를 들어, 숫자나 문자열은 비교가 되니까 가능하지만 우리가 만든 임의의 클래스는 안된다 뭐 이런

### toCollection()

```kotlin
fun <T, C : MutableCollection<in T>> Iterable<T>.toCollection(destination: C): C

```

C는 T를 담을 수 있는 MutableCollection이어야 한다

C : MutableCollection<in T>

→ C는 MutableCollection의 하위 타입인데, 그 컬렉션은 T 타입의 요소를 담을 수 있어야 함

### 클래스 적용

```kotlin
class ListAdapter<T : ItemAdapter> { /* ... */ }

open class ItemAdapter
class MyAdapter : ItemAdapter()

val adapter = ListAdapter<MyAdapter>() // OK

class NotAdapter()
val wrong = ListAdapter<NotAdapter>()  // 컴파일 에러
```

ListAdapter는 오직 ItemAdapter를 상속한 애들만 받을 수 있음

### Any

```kotlin
inline fun <T, R : Any> Iterable<T>.mapNotNull(
    transform: (T) -> R?
): List<R> { ... }

```

코틀린에서 Any는 모든 non-null 타입의 최상위 타입

mapNotNull은 null이 아닌 값들만 모으는 함수이므로 애초에 결과 타입은 null이면 안 됨

### 2개 제한 걸기

```kotlin
fun <T: Animal> pet(animal: T) where T: GoodTempered
```

T는 Animal이면서 동시에 GoodTempered 인터페이스를 구현

Animal이면서 성격이 좋은 애완동물만 받기

이럴 땐 where 절로 확장할 수 있음