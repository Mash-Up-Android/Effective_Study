# **프로퍼티는 동작이 아니라 상태를 나타내야 한다**

코틀린에서 프로퍼티(property)는 객체의 *상태*를 나타내야 하며, *동작*을 표현해서는 안 된다

프로퍼티는 값을 저장하거나 반환하는 역할(상태)만 담당해야 하며, 어떤 계산이나 부수 효과(동작)를 포함해서는 안 된다.

코틀린에서는 아래처럼 getter에 로직을 넣는 것도 가능하다

```kotlin
val nextId: Int
    get() = generateNextId() 
```

호출할 때마다 값이 달라짐

아래처럼 단순 상태만을 나타내자

```kotlin
val name: String = "Alice"  // 단순 상태
var age: Int = 20           // 단순 상태
```

프로퍼티는 객체의 현재 상태를 나타내는 값이어야 하며, 값을 읽을 때마다 결과가 달라지거나 부수 효과가 생기는 동작을 넣지 말아야 한다

동작이 필요한 경우에는 프로퍼티 대신 함수로 구현하는 것이 바람직하다.

예측 가능한 코드 → 유지보수 쉽고, 실수 줄어듦