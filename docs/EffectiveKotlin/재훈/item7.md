# 7. 결과 부족이 발생할 경우 null과 Failure를 사용하라

---

---

# 개요

함수가 원하는 결과를 만들어 낼 수 없는 경우들

- 서버로부터 데이터를 읽어 들이려고 했는데, 인터넷 연결 문제로 읽어 들이지 못한 경우
- 조건에 맞는 첫 번째 요소를 찾으려 했는데, 조건에 맞는 요소가 없는 경우
- 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트의 형식이 맞지 않는 경우

이러한 상황을 처리하는 메커니즘은 크게 다음과 같이 2가지가 있음

- null 또는 실패를 나타내는 sealed 클래스(일반적으로 Failure라는 네이밍)를 리턴
- 예외를 throw

예외는 정보를 전달하는 방법으로 사용돼서는 안되고, 특별한 상황을 나타내고 처리되어야 함

예외는 예외적인 상황이 발생했을 때 사용하는 것이 좋음

이러한 이유는 다음과 같음

- 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못함
- 코틀린의 모든 예외는 unchecked 예외여서 사용자가 예외 처리를 하지 않을 수도 있음
    - unchecked 예외 : 처리하지 않아도 실행에 문제가 없는 예외
    - checked 예외 : 사용자가 반드시 처리하게 강제되는 예외
- 예외는 예외적인 상황을 처리하기 만들어졌으므로 명시적인 테스트(explicit test)만큼 빠르게 동작하지 않음
- try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한됨

충분히 예측할 수 있는 범위의 오류 → null과 sealed class(Failure 네이밍)를 사용

예측하기 어려운 범위의 오류 → 예외를 throw해서 처리

예시

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
    // ...
    if (incorrectSign) {
        return null
    }
    // ...
    return result
}

inline fun <reified T> String.readObject(): Result<T> {
    // ...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    // ...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

이렇게 표시되는 오류는 다루기 쉽고 놓치기 어려움

null을 처리해야 한다면, 안전 호출(safe call)이나 Elvis 연산자 같은 다양한 널 안정성(null- safety) 기능 활용

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

Result와 같은 union type을 리턴할 때는, when을 사용해서 처리

```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
    is Success -> person.age
    is Failure -> -1
 }
```

이러한 오류 처리 방식은 try-catch 블록보다 효율적이며, 사용하기 쉽고 더 명확함

null 값과 sealed result 클래스는 명시적으로 처리해야 하며, 애플리케이션의 흐름을 중지하지도 않음

List 예시

- get : 특정 인덱스에 있는 값을 꺼내는 함수, 해당 인덱스에 없다면 IndexOutOfBoundsException발생
- getOrNull : 특정 인덱스에 없으면 null 리턴

이 외에도 getOrDefault 와 같은 선택지도 있지만, 일반적으로 getOrNull과 Elvis 연산자를 사용하여 예외 처리

개발자는 항상 자신이 요소를 안전하게 추출할 거라 생각하므로 null을 리턴하면 안됨

null이 발생할 수 있다는 경고를 주려면, getOrNull 등을 사용해서 무엇이 리턴되는지 예측할 수 있게 해야함

---