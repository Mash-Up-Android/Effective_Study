# inferred 타입으로 리턴하지 말라

타입 추론은 유명한 코틀린의 특징임

```kotlin
val number = 10 // Int로 추론됨
val message = "Hello, World!" // String으로 추론
fun square(x: Int) = x * x // 반환 타입(Int)을 추론
```

코틀린은 타입 추론(inferred type)을 지원하지만, 반환(return) 타입은 명시적으로 지정하는 것이 더 좋다

왜. 인지 알아보자

inferred 타입은 정확히 오른쪽에 있는 피연산자에 맞게 설정된다

절대 슈퍼클래스, 인터페이스로는 설정되지 않는다

```kotlin
open class Animal
class Zebra: Animal()

var animal = Zebra() // 타입은 Zebra로 추론됨
animal = Animal() // 오류 발생: Type mismatch
```

이러한 제한을 해결하려면 타입을 명시적으로 지정할 수 있다

**`animal`**의 타입을 명시적으로 **`Animal`**로 지정하면 문제가 해결된다

```kotlin
var animal: Animal = Zebra()
animal = Animal() // 정상 작동
```

다음의 인터페이스 예시를 살펴보자

```kotlin
interface CarFactory {
    fun produce(): Car // 반환 타입 명시
}

var DEFAULT_CAR = Fiat126P() // Fiat126P로 추론됨
```

이렇게 작성하면 `CarFactory` 에서는 Fiat126P 이외 자동차를 생산하지 못한다;;

```kotlin
var DEFAULT_CAR : Car() = Fiat126P() // Car로 추론됨
```

앞으로는 외부에서 리턴 타입을 확인할 수 있도록, 타입을 명시적으로 지정해서

유연성과 확장성을 동시에 잡는 코드를 작성해보도록 하자

## **반환 타입을 명확히 명시하자**

- **API 안정성이 높아지고**, 예기치 않은 타입 변경을 방지할 수 있음.

```kotlin
fun getNumbers() = listOf(1, 2, 3) // List<Int>로 추론됨
fun getNumbers() = setOf(1, 2, 3) // Set<Int>로 변경됨
```

다른 코드에서는 여전히 `List`라고 예상하고 있을 수 있음 → API가 깨질 가능성이 생김

- **코드 유지보수가 쉬워지고**, 추론된 타입을 다시 분석할 필요가 없음

```kotlin
fun getData() = mapOf("name" to "Alice", "age" to 25) // Map<String, Any>?
fun getData(): Map<String, Any> = mapOf("name" to "Alice", "age" to 25) // 명시적 타입 지정
```

이 함수가 항상 `Map<String, Any>`을 반환한다는 걸 보장

- **오버라이딩 시 실수할 가능성이 줄어듦.**

```kotlin
open class Parent {
    open fun getMessage() = "Hello" // String으로 추론됨
}

class Child : Parent() {
    override fun getMessage() = 123 // Int로 변경되어 타입 불일치
}

```

**지역 변수의 경우**는 타입을 추론해도 괜찮나?

```kotlin
val name = "아린" // String으로 추론
```