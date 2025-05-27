# 24. 제네릭 타입과 variance 한정자를 활용하라

---

---

# 개요

```kotlin
class Cup<T>
```

위 코드의 타입 파라미터 `T`는 variance 한정자(`out`/`in`)이 없으므로, 기본적으로 invariant(불공변성)임

invariant라는 것은 제네릭 타입으로 만들어지는 타입들이 서로 연광성이 없다는 의미

ex. `Cup<Int>`와 `Cup<Number>`, `Cup<Any>`, `Cup<Nothing>`은 어떠한 관련성도 갖지 않음

```kotlin
fun main() {
    val anys: Cup<Any> = Cup<Int>() // 오류: Type mismatch 
    val nothings: Cup<Nothing> = Cup<Int>() // 오류
}
```

어떤 관련성을 원한다면, variance 한정자인 `out` 또는 `in`을 붙여야 함

### `out`

→ 타입 파라미터를 convariant(공변성)으로 만듦

`A`가 `B`의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 서브타입이라는 의미

```kotlin
class Cup<out T>
open class Dog
class Puppy: Dog()

fun main() { 
    val b: Cup<Dog> = Cup<Puppy>() // OK 
    val a: Cup<Puppy> = Cup<Dog>() // 오류
    val anys: Cup<Any> = Cup<Int>() // OK
    val nothings: Cup<Nothing> = Cup<Int>() // 오류
}
```

### `in`

→ 타입 파라미터를 contravariant(반변성)으로 만듦

`A`가 `B`의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 슈퍼타입이라는 의미

```kotlin
class Cup<in T>
open class Dog
class Puppy: Dog()

fun main() { 
    val b: Cup<Dog> = Cup<Puppy>() // 오류
    val a: Cup<Puppy> = Cup<Dog>() // OK
    val anys: Cup<Any> = Cup<Int>() // 오류
    val nothings: Cup<Nothing> = Cup<Int>() // OK
}
```

---

# 함수 타입

함수타입은 파라미터 유형과 리턴 타입에 따라서 서로 어떤 관계를 가짐

ex. `Int`를 받고, `Any`를 리턴하는 함수를 파라미터로 받는 함수

```kotlin
fun printProcessedNumber(transition: (Int)->Any) { 
    print(transition(42))
}
```

`(Int)->Any` 타입의 함수는 `(Int)->Number`, `(Number)->Any`, `(Number)->Number`, `(Number)->Int` 등으로도 작동함

Kotlin 함수 타입의 모든 파라미터 타입은 contravariant이고, 모든 리턴 타입은 covariant임

파라미터 타입은 Int → Number → Any, 리턴 타입은 Any → Number → Int

---

# variance 한정자의 안정성

자바의 배열은 covatiant임(배열 기반으로 제네릭 연산자는 정렬 함수 등을 만들기 위해)

```java
Integer[] number = {1, 4, 2, 1};
Object[] objects = numbers;
objects[2] = "B" // 런타임 오류: ArrayStoreException
```

위 코드는 컴파일 중에 아무런 문제도 없지만, 런타임 오류가 발생

`numbers`를 `Object[]`로 캐스팅해도 구조 내부에서 사용되는 실질적인 타입은 바뀌지 않음(`Integer`)

따라서 이러한 배열에 `String`타입의 값을 할당하면, 오류가 발생함

코틀린은 이러한 결함을 해결하기 위해서 `Array`(`IntArray`, `CharArray` 등)를 invariant로 만듦

(따라서 `Array<Int>`를 `Array<Any>`등으로 바꿀 수 없음)

파라미터 타입을 예측할 수 있다면, 어떤 서브타입이라도 전달할 수 있음

따라서 아규먼트를 전달할 때, 암묵적으로 업캐스팅할 수 있음

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {}

takeDog(Dog())
takeDog(Puppy())
takeDog(Hound())
```

→ 이는 covarient하지 않음

covarient 타입 파라미터(`out` 한정자)가 `in` 한정자 위치(ex. 타입 파라미터)에 있다면, covariant와 업캐스팅을 연결해서, 우리가 원하는 타입을 아무것이나 전달할 수 있음

즉, value가 매우 구체적인 타입이라 안전한지 않으므로, value를 `Dog`타입으로 지정할 경우, `String`타입을 넣을 수 없음

```kotlin
class Box<out T> { 
    private var value: T? = null // 오류
    
    // Kotlin에서 사용할 수 없는 코드입니다. 
    fun set(value: T) { 
        this.value = value 
    }
    
    fun get(): T = value ?: error("Value not set")
}

val puppyBox = Box<Puppy>()
val dogBox: Box<Dog> = puppyBox
dogBox.set(Hound()) // 하지만 puppy를 위한 공간입니다.

val dogHouse = Box<Dog>()
val box: Box<Any> = dogHouse
box.set("Some String") // 하지만 Dog를 위한 공간입니다.
box.set(42) // 하지만 Dog를 위한 공간입니다.
```

캐스팅 후에 실질적인 객체가 그대로 유지되고, 타이핑 시스템에서만 다르게 처리되기 때문에 이러한 상황은 안전하지 않음(Dog가 들어갈 자리에 Int를 설정하려 하면 오류가 발생)

그래서 Kotlin은 public `in` 한정자 위치(함수 파라미터)에 covariant 타입 파라미터(`out` 한정자)가 오는 것을 금지하여 이러한 상황을 막음

```kotlin
class Box<out T> { 
    var value: T? = null // 오류
    
    fun set(value: T) { // 오류 
        this.value = value 
    }
    
    fun get(): T = value ?: error("Value not set")
}
```

가시성을 private으로 제한하면, 오류가 발생하지 않음

객체 내부에서는 업캐스트 객체에 covaraint(`out` 한정자)를 사용할 수 없기 때문

```kotlin
class Box<out T> { 
    private var value: T? = null
    
    private fun set(value: T) { 
        this.value = value 
    }
    
    fun get(): T = value ?: error("Value not set")
}
```

covariant(`out` 한정자)는 public `out` 한정자 위치에서도 안전하므로 따로 제한되지 않음

이러한 안정성의 이유로 생성되거나 노출되는 타입에만 covariant(`out` 한정자)를 사용하는 것임

이러한 프로퍼티는 일반적으로 `producer` 또는 `immutable` 데이터 홀더에 많이 사용됨

좋은 예로 `T`는 convariant인 `List<T>`가 있음

앞서 설명한 이유로 함수의 파라미터가 `List<Any?>`로 예측된다면, 별도의 변환 없이 모든 종류를 파라미터로 전달할 수 있음

다만 `MutableList<T>`에서 `T`는 `in` 한정자 위치에서 사용되며, 안전하지 않으므로 invariant임

```kotlin
fun append(list: MutableList<Any>) { 
    list.add(42)
}

val strs = mutableListOf<String>("A", "B", "C")
append(strs) // Kotlin에서 사용할 수 없는 코드
val str: String = strs[3]
print(str)
```

다른 좋은 예로는 `Reponse`가 있는데, `Response`를 사용하면 다양한 이득을 얻을 수 있음

variance 한정자 덕분에 이 내용은 모두 참이 됨

- `Response<T>`라면 `T`의 모든 서브타입이 허용됨
    
    ex. `Response<Any>`가 예상된다면, `Response<Int>`와 `Response<String>`이 허용됨
    
- `Response<T1, T2>`라면 `T1`과 `T2`의 모든 서브타입이 허용됨
- `Failure<T>`라면, `T`의 모든 서브타입 `Failure`가 허용된다
    
    ex. `Failure<Number>`라면, `Failure<Int>`와 `Failure<Double>`이 모두 허용됨
    
    ex. `Failure<Any>`라면, `Failure<Int>`와 `Failure<String>`이 모두 허용됨
    
- convariant와 `Nothing` 타입으로 인해서 `Failure`는 오류 타입을 지정하지 않아도 되고, `Success`는 잠재적인 값을 지정하지 않아도 됨
    
    ```kotlin
    sealed class Response<out R, out E>
    class Failure<out E>(val error: E): Response<Nothing, E>()
    class Success<out R>(val value: R): Response<R, Nothing>()
    ```
    

covariant와 public `in` 위치와 같은 문제는 contravariant 타입 파라미터(`in` 한정자)와 public `out` 위치(함수 리턴 타입 또는 프로퍼티 타입)에서도 발생함

`out` 위치는 암묵적인 업캐스팅을 허용함

```kotlin
open class Car
interface Boat
class Amphibious: Car(), Boat

fun getAmphibious(): Amphibious = Amphibious()

val car: Car = getAmphibious()
val boat: Boat = getAmphibious()
```

사실 이는 contravariant(`in` 한정자)에 맞는 동작이 아님

다음 코드를 보면, 어떤 상자에 어떤 타입이 들어 있는지 확실하게 알 수 없음

```kotlin
class Box<in T>(
    // 코틀린에서는 사용할 수 없는 코드입니다.
    val value: T
)

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
val boat: Boat = garage.value // 하지만 Car를 위한 공간입니다.

val noSpot: Box<Nothing> = Box<Car>(Car())
val boat: Nothing = noSopt.value // 아무것도 만들 수 없습니다.
```

이러한 상황을 막기 위해, 코틀린은 contravariant 타입 파라미터(`in` 한정자)를 public `out` 한정자 위치에 사용하는 것을 금지하고 있음

```kotlin
class Box<in T> { 
    var value: T? = null // Error

    fun set(value: T) {
    	this.value = value
    }

    fun get(): T = value // Error
    	?: error("Value not set")
}
```

이번에도 요소가 private일때는 아무 문제 없음

```kotlin
class Box<out T> { 
    private var value: T? = null

    fun set(value: T) {
    	this.value = value
    }

    private fun get(): T = value
    	?: error("Value not set")
}
```

이런 형태로 타입 파라미터에 contravariant(`in` 한정자)를 사용함

---

# variance 한정자의 위치

variance 한정자는 크게 두 위치에 사용할 수 있음

1. 선언 부분

- 일반적으로 이 위치에 사용
- 이 위치에서 사용하면 클래스와 인터페이스 선언에 한정자가 적용됨
    
    ⇒ 클래스와 인터페이스가 사용되는 모든 곳에 영향을 줌
    

```kotlin
// 선언 쪽의 variance 한정자
class Box<out T>(val value: T)
val boxAny: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```

2. 클래스와 인터페이스를 활용하는 위치

- 특정한 변수에만 variance 한정자가 적용됨

```kotlin
class Box<T>(val value: T)
val boxAny: Box<String> = Box("Str")
// 사용하는 쪽의 variance 한정자
val boxAny: Box<out Any> = boxStr
```

모든 인스턴스에 variance 한정자를 적용하면 안 되고, 특정 인스턴스에만 적용해야 할 때 이런 코드를 사용

ex. `MutableList`에 `in` 한정자를 포함하면, 요소를 리턴할 수 없으므로 `in` 한정자를 붙이지 않음

하지만 단일 파라미터 타입에 `in` 한정자를 붙여서 contravariant를 가지게 하는 것은 가능함

→ 이렇게 하면 여러가지 타입을 받아들이게 가능

variance 한정자를 사용하면 위치가 제한될 수 있음

- ex. `MutableList<out T>`가 있다면, `get`으로 요소 추출 시 `T`타입이 리턴되지만, `set`은 `Nothing` 타입의 아규먼트가 전달될 거라 예상되므로 사용할 수 없음
    
    → 모든 타입의 서브타입을 가진 리스트(`Nothing` 리스트)가 존재할 가능성이 있기 때문
    
- ex. `MutableList<in T>`를 사용할 경우, `get`과 `set`을 모두 사용할 수 있지만, `get`을 사용할 경우 전달되는 자료형은 `Any?`임
    
    → 모든 타입의 슈퍼타입을 가진 리스트(`Any` 리스트)가 존재할 가능성이 있기 때문
    

---

# 정리

코틀린은 타입 아규먼트의 관계에 제약을 걸 수 있는 굉장히 강력한 제너릭 기능을 제공함

이러한 기능으로 제네릭 객체를 연산할 때 굉장히 다양한 지원을 받을 수 있음

코틀린에는 다음과 같은 타입 한정자가 있음

- 타입 파라미터의 기본적인 variance 동작은 invariance임
    - `A`가 `B`의 서브타입이라고 할 때, `Cup<A`와 `Cup<B>`는 아무런 관계를 갖지 않음
- `out` 한정자는 타입 파라미터를 covariant하게 만듦
    - `A`가 `B`의 서브타입이라고 할 때, `Cup<A>`는 `Cup<B>`의 서브 타입이 됨
- `in` 한정자는 타입 파라미터를 contravariant하게 만듦
    - `A`가 `B`의 서브타입이라고 할 때, `Cup<B>`는 `Cup<A>`의 슈퍼 타입이 됨

코틀린에서는

- `List`와 `Set`의 타입 파라미터는 covariant(`out` 한정자)임
    
    `List<Any>`가 예상되는 모든 곳에 전달 가능
    
    `Array`, `MutableList`, `MutableSet`, `MutableMap`의 타입 파라미터는 invariant임
    
- 함수 타입의 파라미터 타입은 contravariant(`in` 한정자)이고, 리턴 타입은 covariant(`out` 한정자)임
- 리턴만 되는 타입에는 covariant(`out` 한정자)를 사용
- 허용만 되는 타입에는 contravariant(`in` 한정자)를 사용

---