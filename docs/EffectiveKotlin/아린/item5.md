# 예외를 활용해 코드에 제한을 걸어라

## 제한을 걸면 다음과 같은 장점이 있다

- 문서를 읽지 않은 개발자도 문제를 확인할 수 았다
- 문제가 있으면 예상치 못한 동작을 하지 않고 예외를 throw함
    - 예외를 던지는 것이 예상치 못한 동작을 하는 것보다 훨씬 안전함
- 어느정도 자체 검사가 되어서 단위 테스트 범위가 줄어든다
- 스마트 캐스트가 가능해져서 타입 변환을 적게 할 수 있다

# 제한을 거는 4가지 방법

## 1. `require` 블록 : 아규먼트를 제한한다

제한을 만족하지 못할 경우 예외를 throw 한다

### 아규먼트 VS 파라미터

```kotlin
fun exam(x: Int, y: Int) {
		return x+y
}
```

여기서 x, y가 파라미터고

x, y에 1, 2를 대입하면 1, 2를 아규먼트 라고 한다

⇒ x, y라는 파라미터에 1, 2라는 아규먼트를 대입

Int 타입을 아규먼트로 받아서 숫자가 양의 정수인지 확인하는 코드

```kotlin
fun factorial(n: Int) : Long {
		require(n >= 0)
		return if(n <= 1) 
				else factorial(n-1) * n
}
```

코드 가장 앞부분에 유효성 검사 코드를 명시함으로써, 읽는 사람도 쉽게 확인할 수 있도록 한다

하지만 코드를 읽지 않는 사람이 있을 수 있으므로 문서에 명시에 두긴 해야한다

require 함수는 조건을 만족하지 못할 때 무조건 `illegalargument exception`을 발생시킴.

### `illegalargument exception`

- 메서드에 전달된 파라미터가 예상된 타입에서 벗어나는 경우

람다를 활용해서 지연 메시지를 설정할 수도 있다

```kotlin
fun factorial(n: Int) : Long {
		require(n >= 0) {
				"$n 은 양의 정수여야 합니다"
		}
		return if(n <= 1) 
				else factorial(n-1) * n
}
```

## 2. `check` 블록 : 상태와 관련된 동작을 제한한다

어떤 구체적인 조건을 만족할 때만 상태를 사용할 수 있게 한다

- 객체가 초기화되어 있는 경우만 처리

```kotlin
fun speak(text: String) {
		check(isInitialized)
}
```

- 토큰이 있는 경우만 처리

```kotlin
fun getUserInfo(): UserInfo {
		checkNotNull(token)
}
```

check와 require 함수가 다른 점은

check 함수는 IllegalStateException 예외를 throw 한다는 점이다

또힌, 함수 전체에 대한 어떤 예측을 검증할 떄 일반적으로 require를 배치한다

즉, require 뒤에 check가 나오는 것이 일반적이다

사용자가 코드를 제대로 사용할 거라 믿지 말고 항상 의심하고 체크하여, 문제 상황에 예외를 던지자.

이를 스스로 검증할 때에는 assert 계열 함수를 활용한다



## 3. `assert` 블록 : 테스트 모드에서 어떤 것이  true인지 확인한다

예를 들어, 어떤 함수가 10개의 요소를 리턴한다고 하면

이 함수가 진짜 10개의 요소를 리턴 하는지에 대한 테스트 코드는 true일 것이다

하지만 이 함수가 제대로 구현되어 있지 않을 때 발생할 문제를 예방하기 위해 **단위 테스트**를 한다

```kotlin
class StackTest {
		@Test
		fun 'Stack pops correct number of elements'() {
				val stack = Stack(20) { it }
				val ret = stack.pop(10)
				
				assertEquals(10, ret.size)
		}
}
```

이 테스트 코드에서는 한 경우만 테스트하게 된다.

하지만 모든 pop 호출 위치에서 제대로 pop이 동작하는지 확인해보는 것이 더욱 신뢰도가 높아질 것이다

```kotlin
fun pop(num: Int = 1): List<T> {
		assert(ret.size == num)
		return ret
}
```

## 4. `nullability`와 스마트 캐스팅

- 어떤 사람의 복장이 드레스일 때 changeDress가 실행됨

```kotlin
fun changeDress(person: Person) {
		require(person.outfit is Dress)
		val dress: Dress = person.outfit
}
```

- 어떤 대상이 null인지 확인하는 코드

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
		require(person.email != null)
		val email: String = person.email
}
```

- `requireNotNull`, `checkNotNull` 함수를 사용해도 된다

```kotlin
class Person(val email: String?)
fun validateEmail(email: String?) { }

fun sendEmail(person: Person, message: String) {
		val email = requireNotNull(person.email)
		validateEmail(email)
}

fun sendEmail(person: Person, text: String) {
		requireNotNull(person.email)
		validateEmail(person.email)
}
```

- nullability로 Elvis 연산자 활용도 가능

```kotlin
fun sendEmail(person: Person, message: String) {
		val email: String = person.email ?: return
}
```

- 프로퍼티에 대한 null 처리가 여러 개일 때, `return`/`throw`와 `run` 함수 사용

```kotlin
fun sendEmail(person: Person, text: String) {
		val email: String = person.email ?: run {
				log("이메일 주소가 없습니다")
				return
		}
}
```

이렇게 하면 함수 중지 이유를 로그에 출력할 수 있다

return, throw를 활용한 Elvis 연산자로 nullable을 확인하는 것은 굉장히 관용적인 방법이다.

코드 앞부분에서 적극적으로 사용하도록 하자