variance 한정자는 in out을 의미

이런게 안붙어있는 제네릭은 불공변성(invariant)이라고 부름

불공변성은 제네릭으로 만들어지는 타입들이 관련이 없다~라는 뜻

관련을 갖게 하려면 variance 한정자를 붙여~

> Q. 나는 이 관련성이 없다는 말부터가 정확히 어떤건지 잘 모르겠넹?

### variance 한정자

일단 `class Cup<in T>()` `class Cup<out T>()` 이런거 붙인게 variance 한정자를 붙인거

out을 붙이게되면 공변성~ 코배리언트(covariant)

`A`가 `B`의 서브타입일 때 `Cup<A>`가 `Cup<B>`의 서브타입이다.

```kotlin
class Cup<out T>()
open class Dog
class Puppy : Dog()

fun main() {
    val a: Cup<Dog> = Cup<Puppy>()
    val b: Cup<Puppy> = Cup<Dog>()  // error
}
```

이렇게 되면 `Puppy`가 `Dog`의 서브타입이니까 `Cup<Puppy>`가 `Cup<Dog>`의 서브타입임이 가능

in을 붙이게되면 반변성~ 콘트라배리언트 (contravariant)

`A`가 `B`의 서브타입일 때 `Cup<B>`가 `Cup<A>`의 서브타입이다

```kotlin
class Cup<in T>()
open class Dog
class Puppy : Dog()

fun main() {
    val a: Cup<Dog> = Cup<Puppy>()  // error
    val b: Cup<Puppy> = Cup<Dog>()
}
```

이렇게 되면 `Puppy`가 `Dog`의 서브타입이니까 `Cup<Dog>`가 `Cup<Puppy>`의 서브타입임이 가능

애초에 이런 코드자체가 작성이 불가능한데

```kotlin
class Box<T>()

val c: Box<Int> = Box<Number>()  // class Box<in T>()
val d: Box<Number> = Box<Int>()  // class Box<out T>()
```

in out을 붙이게되면 동작이 잘 될거다~

> Q. in, out을 어떨때 썼는지 잘 기억이가 안나넹?

### 함수타입

함수타입 람다에 제네릭이 들어간다면?

`(Int) → Any`라고 되어있으면 `Any`가 상위타입이기 때문에 어떤걸 해도 상관없음, `Int`는 상위타입으로 가능

- `(Int) → Any`
    - `(Int) → Number`, `(Int) → Int`, …
    - `(Number) → Any`, `(Number) → Number`, `(Any) → Int`가 가능

`Any?`는 `Any`의 상위타입으로 동작

> Q. Nothing?이 Any의 상위타입? 에이 구라 ㅋ Nothing이 모든 타입의 최하위 타입인데 어케 이래되누!

예측가능한 파라미터 타입은 어떤 서브타입이라도 전달이 가능