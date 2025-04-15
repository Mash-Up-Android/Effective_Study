# 4. inffered 타입으로 리턴하지 말라

---

---

# 개요

- 코틀린 타입 추론(type inference)을 사용할 때는 몇 가지 위험한 부분이 있음
- 이러한 위험한 부분을 피하려면, 할당 때 inferred 타입은 정확하게 오른쪽에 있는 피연산에 맞게 설정된다는 것을 기억해야 함
- 절대 슈퍼 클래스 또는 인터페이스로는 설정되지 않음

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal = Zebra()
    animal = Animal()
}
```

일반적인 경우 이러한 것들이 문제가 되지 않음

원하는 타입보다 제한된 타입이 설정되었다면, **타입을 명시적으로 지정**해서 해결 가능

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal: Animal = Zebra()
    animal = Animal()
}
```

하지만 직접 라이브러리(또는 모듈)를 조작할 수 없는 경우에는 이런 문제를 간단하게 해결할 수 없음

이러한 경우에서 inferred 타입을 노출하면 위험한 일이 발생할 수 있음

다음과 같이 CarFactory 인터페이스가 있다고 가정

```kotlin
interface CarFactory {
    fun produce(): Car
}

val DEFAULT_CAR: Car = Flat126P()
```

DEFAULT_CAR는 Car로 명시적으로 지정되어 있으므로 메소드 리턴 타입을 제거

```kotlin
interface CarFactory {
    fun produce() = DEFAULT_CAR
}
```

그러나 이후 다른 사람이 코드를 보다가, DEFAULT_CAR 는 타입 추론에 의해 자동으로 타입이 지정될 것이므로, Car를 명시적으로 지정하지 않아도 된다고 생각해서, 지정한 코드를 제거 하게 된다면 CarFactory는 Flat126P 이외의 자동차를 생산할 수 없게 됨

```kotlin
val DEFAULT_CAR = Flat126P()
```

만약 인터페이스를 직접 만들었다면, 문제를 쉽게 찾아서 수정할 수 있지만, 외부 API라면 문제를 쉽게 해결할 수 없기 때문에, 리턴 타입을 명시적으로 지정해주는게 좋음

---

# 정리

- 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정
- 안전을 위해 외부 API를 만들 때는 반드시 타입을 지정하고 이렇게 지정한 타입을 특별한 이유와 확실한 확인 없이 제거하지 않는 것이 좋음
- inferred 타입은 프로젝트가 진전될 때, 제헌이 너무 많아지거나 예측하지 못하는 결과를 낼 수 있음

---