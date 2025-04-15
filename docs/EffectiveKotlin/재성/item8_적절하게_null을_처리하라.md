### null은 명확하게 의미를 갖는게 좋다.

이미 Kotlin 문법적으로 이를 잘 보여주고 있네? (firstOrNull, getOrNull, 뭐시기 널널)

### Null을 안전하게 처리하자!

Elvis연산자를 통해 리턴을 때리던 예외를 던지던 하자!

컬렉션에서는 orEmpty를 지원한다.

Non-null Assertion을 피하자. 어떤 시점에 코드가 어떻게 변경될지 예측하기도 힘들고 Null일 가능성을 아예 배제시키는 것

### 방어적 프로그래밍 vs 공격적 프로그래밍

null을 처리하고 최대한 안정적으로 돌아가게 하는게 방어적 프로그래밍

require, check 때리고 하면 공격적 프로그래밍

requireNotNull이나 checkNotNull을 통해 에러가 남을 알려주는게 좋을 때도 있음

### 의미없는 Nullability를 피하자

null은 결국에 처리해야 하기 때문에 이또한 비용

의미없는 null은 최대한 없애는게 좋다

이걸 못없애면 나중에 다른 개발자가 Non-null Assertion을 쓸수도 있음

List<Int>?를 쓰지말자!

### lateinit과 Delegates notNull

lateinit은 프로퍼티를 지연 초기화 할 때 쓰는데, 사용하는 시점에 초기화가 되어있음을 보장하고 지연초기화 하는 것 (초기화가 안되어있으면 예외 던짐)

Primitive Type은 어떻게 할까? 이때 사용하는게 Delegates notNull

```kotlin
private var id: Int by Delegates.notNull()
```

이런식으로 위임을 하게 되면 Nullability로 발생할 수 있는 문제를 예방할 수 있음