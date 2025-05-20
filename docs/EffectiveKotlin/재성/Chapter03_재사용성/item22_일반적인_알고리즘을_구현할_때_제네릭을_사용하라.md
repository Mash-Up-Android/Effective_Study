타입 아규먼트를 갖는 함수를 제네릭 함수라고 부른다? 몰랐누

```kotlin
fun <T> identify(value: T): T = value
```

제네릭 타입은 컴파일타임에 사라지지만, 개발 과정에서 특정 타입을 사용하도록 제한할 수 있다.

타입 파라미터의 중요한 점은 구체타입의 서브타입만 사용할 수 있도록 제한하는 것

예를 들어서 Comparable타입으로 타입 제한을 걸어버리면 Comparable에서 제공하는 함수들을 사용 가능하다~

우리가 Any를 받게되면 nullable하지 않은 타입들로만 제한거는 거랑 똑같기 때문

왜냐? Any?가 Any의 상위 타입이기 때문

요런 코드는 또 첨봤네? 이런거 써봄?

```kotlin
fun <T: Animal> pet(animal: T) where T: GoodTempered { /* */ }
```