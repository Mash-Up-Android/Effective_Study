### 예외를 발생시키는 방법

1. require : argument에 제한을 둘 수 있음 (IllegalArgumentException)
2. check : 상태 관련 동작에 제한을 둘 수 있음 (IllegalStateException)
3. assert : Test 환경에서 조건을 만족하도록 제한을 둘 수 있음 (AssertionError)
4. elvis return/throw

위 함수를 사용했을 때 장점?이 정말 좋다고 생각

코드를 읽을 때 어떤 경우에 발생할 수 있는지 미리 문제 파악이 가능하고, 예상치 못한 동작을 미리 제어할 수 있음

### 스마트 캐스팅 및 null check에 용이하다?

```kotlin
fun changeDress(person: Person) {
		require (person.outfit is Dress)
		val dress: Dress = person.outfit  // Dress로 스마트캐스트 됨
}
```

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
		require (person.email != null)  // null check를 할 수 있음, requireNotNull
		val email: String = person.email
}
```

### Elvis 잘 사용하기

require나 check를 해서 throw 하는 것보다 return이 더 안정적이라고 생각되긴 함

```kotlin
fun sendEmail(person: Person, text: String) {
		val email: String = person.email ?: return
		
		val email: String = person.email?.run {
				log("널이다~@~@~@")
				return
		}
}
```