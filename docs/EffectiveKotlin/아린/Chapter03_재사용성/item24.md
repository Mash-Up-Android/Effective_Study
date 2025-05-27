# 제네릭 타입과 variance 한정자를 활용하라

```kotlin
class Cup<T>
```

여기서의 타입 파라미터 T는 variance 한정자인 `out`이나 `in`이 없다

이러한 상황에서 T는 불공변성(invariant)이다

불공변성이란, 제네릭 타입으로 만들어지는 타입들이 아무런 연관이 없다는 것

```kotlin
fun main() {
		val anys : Cup<Any> = Cup<Int>() // Type mismatch
		val nothings : Cup<Nothing> = Cup<Int> // Type mismatch
}
```

이렇게 Cup<Any>, Cup<Int>, Cup<Nothing>, Cup<Int>는 어떤 연관도 없다

이들에게 어떤 관련성을 원한다면 variance 한정자를 활용하면 된다

### out은 타입 파라미터를 공변성(covariant)으로 만든다

```kotlin
Class Cup<out T>
open class Dog
class Puppy : Dog()

fun main(args: Array<String>) {
		val b : Cup<Dog> = Cup<Puppy> // OK
		val a : Cup<Puppy> = Cup<Dog> // 오류

		val anys : Cup<Any> = Cup<Int>() // OK
		val nothings : Cup<Nothing> = Cup<Int>() // 오류
}
```

이 코드에서 out이라는 타입 파라미터로 인해
Puppy가 Dog의 `서브타입`일 때 Cup<Puppy>는 Cup<Dog>의 `서브타입`이 된다

### in은 타입 파라미터를 반변성(contravariant)로 만든다

```kotlin
Class Cup<in T>
open class Dog
class Puppy : Dog()

fun main(args: Array<String>) {
		val b : Cup<Dog> = Cup<Puppy> // 오류
		val a : Cup<Puppy> = Cup<Dog> // OK

		val anys : Cup<Any> = Cup<Int>() // 오류
		val nothings : Cup<Nothing> = Cup<Int>() // OK
	}
```

Puppy가 Dog의 `서브타입`일 때 Cup<Puppy>는 Cup<Dog>의 `슈퍼타입`이 된다

이 이후로… [함수타입] 부분;; 모든 그림과 그래프 표 모두 이해가 안되었음 너무 어려워서 한계를 느낌 (아린 : ㅠㅠ)

## variance 한정자의 위치

- 리턴만 되면 out 한정자를 사용한다
    - `out T` → **T를 반환(출력)** 하는 경우 사용 (읽기만 가능, 넣기는 불가)
- 허용만 되면 in 한정자를 사용한다
    - `in T` → **T를 파라미터(입력)** 로 받는 경우 사용 (쓰기 가능, 반환은 불가)