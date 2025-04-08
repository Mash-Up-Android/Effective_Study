# 5. 예외를 활용해 코드에 제한을 걸어라

---

---

# 개요

코틀린에서 예외를 활용해 동작에 제한을 거는 방법

- require : 아규먼트를 제한
- check : 상태와 관련된 동작을 제한
- assert : 어떤 것이 true인지 확인(테스트 모드에서만 작동)
- Elvis 연산자(?:) : return 또는 throw와 함께 활용하여 제한

```kotlin
// Stack<T> 일부
fun pop(num: Int = 1): List<T> {
    require(num <= size) {
        "Cannot remove more elements than current size"
    }
    check(isOpen) { "Cannot pop from closed stack" }
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)
    return ret
}
```

이렇게 제한을 걸어 주면 다양한 장점이 발생

- 제한을 걸면 문서를 읽지 않은 개발자도 문제를 확인 가능
- 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고, 예외를 throw함(예상하지 못한 동작을 하는 것은 예외를 throw하는 것보다 위험하고 상태를 관리하기 힘듦 ⇒ 코드가 더 안정적으로 작동)
- 코드가 어느정도 자체적으로 검사됨 ⇒ 단위 테스트를 줄임
- 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트(타입 변환)을 적게 할 수 있음

---

# 아규먼트

함수를 정의할 때 타입 시스템을 활용해서 아규먼트(argument)에 제한을 거는 코드를 많이 사용

- 예시
    - 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야 함
    - 좌표들을 아규먼트로 받아서 클러스터를 찾을 때는 비어 있지 않은 좌표 목록이 필요
    - 사용자로부터 이메일 주소를 입력받을 때는 값이 입력되어 있는지, 이메일 형식이 올바른지 확인

일반적으로 이러한 제한을 걸 때는 **require 함수**를 사용

require : 제한을 확인하고, 제한을 만족하지 못할 경우 예외를 throw함

```kotlin
// require 내부 구현
public inline fun require(value: Boolean, lazyMessage: () -> Any): Unit {
    contract {
        returns() implies value
    }
    if (!value) {
        val message = lazyMessage()
        throw IllegalArgumentException(message.toString())
    }
}
```

이와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로, 읽는 사람도 쉽게 확인 가능

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    //...
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    //...
}
```

require는 조건을 만족하지 못할 때  `IllegalArgumentException`을 발생시키므로 제한을 무시할 수 없음

일반적으로 이러한 처리는 함수의 가장 앞부분에 하게 되므로, 코드를 읽을 때 쉽게 확인 가능

다음과 같은 방법으로 람다를 활용해서 지연 메시지를 정의 가능

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0) {
        "Cannot calculate factorial of $n because it is smaller than 0"
    }
    return if (n <= 1) 1 else factorial(n - 1) * n
}
```

---

# 상태

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 하는 경우

- 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수
- 사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

```kotlin
// check 내부 구현
public inline fun check(value: Boolean, lazyMessage: () -> Any): Unit {
    contract {
        returns() implies value
    }
    if (!value) {
        val message = lazyMessage()
        throw IllegalStateException(message.toString())
    }
}
```

상태와 관련된 제한을 걸 때는 일반적으로 check 함수를 사용하여 상태가 올바른지 확인

```kotlin
fun speak(text: String) {
    check(isInitialized)
    //...
}

fun getUserInfo(): UserInfo {
    checkNotNull(token)
    //...
}

fun next(): T {
    check(isOpen)
    //...
}
```

check 함수는 require와 비슷하지만, 지정된 조건을 만족하지 못하면 `IllegalStateException`을 throw함

예외 메시지는 require와 마찬가지로 지연 메시지를 사용해서 변경 가능

함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 require 블록 뒤에 check를 배치

---

# Assert 계열 함수 사용

함수가 올바르게 구현되지 않아서 발생할 수 있는 문제를 예방하기 위해서는 단위 테스트를 사용

```kotlin
// assert 내부 구현
public inline fun assert(value: Boolean, lazyMessage: () -> Any) {
    if (_Assertions.ENABLED) {
        if (!value) {
            val message = lazyMessage()
            throw AssertionError(message)
        }
    }
}
```

예시

```kotlin
class StackTest {
    @Test
    fun 'Stack pops correct number of elements'() {
        val stack = Stack(20) { it }
        val ret = stack.pop(10)
        assertEquals(10, ret.size)
    }

    // ...
}
```

위 코드에서 스택이 10개인 요소를 pop하면, 10개의 요소가 나온다는 보편적인 사실을 테스트하지만, 모든 상황에서 괜찮은지 알 수 없으므로 모든 pop 호출 위치에서 제대로 동작하는지 확인하는 것이 좋음

```kotlin
fun pop(num: Int = 1): List<T> { 
    //... 
    assert(ret.size == num)
    return ret
}
```

이러한 조건은 현재 Kotlin/JVM에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인 가능

프로덕션 환경에서는 오류가 발생하지 않으므로, 심각한 결과를 초래할 수 있는 경우에는 check를 사용해야 함

단위 테스트 대신 함수에서 assert를 사용하면 다음과 같은 장점이 있음

- Assert 계열의 함수는 코드를 자체 점검하며, 더 효율적으로 테스트할 수 있게 해줌
- 특정 상황이 아닌 모든 상황에 대한 테스트 가능
- 실행 시점에 정확하게 어떻게 되는지 확인 가능
- 실제 코드가 더 빠른 시점에 실패하게 만들어서, 예상하지 못한 동작이 언제 어디서 실행되었는지 쉽게 찾음

---

# nullability와 스마트 캐스팅

require와 check 블록으로 어떤 조건을 확인해서 true가 나왔다면, 해당 조건은 이후에도 true일 거라고 가정

따라서 이를 활용하여 타입 비교를 했다면, 스마트 캐스트가 작동

```kotlin
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
    // ...
}
```

outfit 프로퍼티가 final이라면 Dress로 스마트 캐스트됨

이러한 특정은 어떤 대상이 null인지 확인할 때 유용

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
    require(person.email != null)
    val email: String = person.email
    // ...
}
```

`requireNotNull`, `checkNotNull`이라는 특수한 함수도 있음

둘 다 스마트 캐스트를 지원하므로, 변수를 언팩(unpack)하는 용도로 활용 가능

```kotlin
class Person(val email: String?)
fun validateEmail(email: String) {//...}

fun sendEmail(person: Person, text: String) {
    val email = requireNotNull(person.email)
    validateEmail(email)
    //...
}

fun sendEmail(person: Person, text: String) {
    requireNotNull(person.email)
    validateEmail(person.email)
    //...
}
```

nullability를 목적으로, 오른쪽에 throw또는 return을 두고 Elvis 연산자를 활용 가능

이러한 코드는 굉장히 읽기 쉽고, 유연하게 사용 가능

```kotlin
fun send(person: Person, text: String) {
    val email: String = person.email ?: return
    // ...
}
```

run함수를 조합해서 로그 출력도 가능

```kotlin
fun sendEmail(perosn: Person, text: String) {
    val name: String = person.name ?: return
    val email: String = person.email ?: run {
        log("Email not sent, no email address")
        return
    }
    //...
}
```

---

# 정리

이와 같은 내용을 기반으로, 다음과 같은 이득을 얻을 수 있음

- 제한을 훨씬 더 쉽게 확인 가능
- 애플리케이션을 더 안정적으로 지킬 수 있음
- 코드를 잘못 쓰는 상황을 막을 수 있음
- 스마트 캐스팅을 활용 가능

이를 위해 활용했던 메커니즘을 정리하면 다음과 같음

- require 블록 : 아규먼트와 관련된 예측을 정의할 때 사용
- check 블록 : 상태와 관련된 예측을 정의할 때 사용
- assert 블록 : 테스트 모드에서 테스트를 할 때 사용
- return과 throw와 함께 Elvis 연산자 사용

이외에도 다른 오류들을 발생시킬 때 throw를 활용 가능

---