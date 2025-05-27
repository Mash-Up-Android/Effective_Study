# **Generics**

Kotlin의 클래스는 타입 파라미터를 가질 수 있다

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

이러한 클래스의 인스턴스를 생성하려면 간단히 타입 아규먼트를 제공하면 된다.

```kotlin
val box: Box<Int> = Box<Int>(1)
```

하지만 매개변수를 생성자 인수 등에서 유추할 수 있는 경우 타입 아규먼트는 생략할 수 있다.

```kotlin
val box = Box(1)
```

# **Variance**

Java에서 제네릭 타입 `List<String>` 과 `List<Object>` 는 서로 별개의 타입

```java
List<String> strs = List.of("a","b");
// List<Object> objs = strs;          // 에러
List<? extends Object> objs = strs;   // 가능 (읽기 전용)
```

`List<String>` 을 `List<Object>` 변수에 바로 넣을 수 없어서 `List<? extends Object>` (covariant) 또는 `List<? super String>` (contravariant) 같은 와일드카드를 사용

코틀린은 자바의 `? extends`, `? super` 같은 와일드 카드 타입이 없는 대신,
선언 지점 변성과 타입 프로젝션이 있다

### 선언 지점 변성(Declaration-site variance)

클래스를 만들 때 **선언 지점**에 `out` 또는 `in` 을 붙여서 변성을 미리 지정한다

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs 
}
```

- `out T` → T를 내보내기만(produce) 함. → `Source<String>` 은 `Source<Any>` 의 하위 타입
- `Array<out Any>` → 읽기만 가능(Producer)
- 선언부에서 “out” 붙여두면, 이 타입은 공변(covariant) 타입이 됨

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0)
    val y: Comparable<Double> = x
}
```

- `in T` → T를 받기만(consume) 함. → `Comparable<Number>` 는 `Comparable<Double>` 의 하위 타입
- `Array<in String>` → 쓰기만 가능(Consumer)
- in은 타입 매개변수를 반공변적(contravariant)으로 만든다

### 타입 프로젝션(Type projection)

선언부에서 변성을 미리 지정할 수 없을 때( Array<T>처럼 get, set 둘 다 해야 하는 경우 )
**사용 지점**에서 `out` / `in`을 붙여서 와일드카드와 같은 효과를 내는 방법

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
    // from은 “읽기 전용”으로 제한
    for (i in from.indices) {
        to[i] = from[i]
    }
}

fun fill(dest: Array<in String>, value: String) {
    // dest는 “쓰기 전용”으로 제한
    for (i in dest.indices) {
        dest[i] = value
    }
}

```

### 스타 프로젝션 (**Star-projections)**

타입 인자를 모르지만 “안전하게” 사용하고 싶을 때

```kotlin
interface Function<in T, out U>
Function<*, String>   // Function<in Nothing, String>
Function<Int, *>      // Function<Int, out Any?>
Function<*, *>        // Function<in Nothing, out Any?>
```