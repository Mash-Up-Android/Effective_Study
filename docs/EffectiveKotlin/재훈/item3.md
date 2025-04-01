# 3. 최대한 플랫폼 타입을 사용하지 말라

---

---

# 개요

- 코틀린에서 널 포인터 예외(NPE)는 null-safety 매커니즘으로 인해 거의 찾아보기 힘듦
- null-safety 매커니즘이 없는 자바, C 등의 언어와 코틀린을 연결해서 사용할 때는 NPE 예외가 발생 가능

자바 코드에서 `@Nullable` 어노테이션이 붙어 있다면 이를 nullable로 추정하여 `String?`으로 변경하고, `@NotNull` 어노테이션이 붙어 있다면, `String`으로 변경하면 되는데,

```java
public class JavaTest{
		public String giveName() {
				// ...
		}
}
```

만약 위 자바 코드처럼 어노테이션이 붙어 있지 않다면, 자바에서는 모든 것이 nullable일 수 있으므로 최대한 안전하게 접근하기 위해 nullable로 가정하고 다뤄야 함

### **제네릭 타입**

```java
public class UserRepo {
		public List<User> getUsers() {
				// ...
		}
}
```

```kotlin
val users:List<User> = UserRepo().users!!.filterNotNull()
```

코틀린이 디폴트로 모든 타입을 nullable로 다룬다면, 이를 사용할 때 이러한 리스트와 리스트 내부의 User 객체들이 널 아니라는 것을 알아야 함

그래서 코틀린은 자바 등의 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루고, 이러한 타입을 **플랫폼 타입**이라고 부름

플랫폼 타입은 String! 처럼 타입 이름 뒤에 ! 기호를 붙여서 표기함(직접적으로 코드에 나타나지는 않음)

```kotlin
val repo = UserRepo()
val user1 = repo.user       // user1의 타입은 User!
val user2: User = repo.user // user2의 타입은 User
val user3: User? = repo     // user3의 타입은 User?

val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```

- 문제는 null이 아니라고 생각되는 것이 null일 가능성이 있으므로 여전히 위험 ⇒ 항상 주의를 기울여야 함
- 설계자가 명시적으로 어노테이션으로 표기하거나 주석으로 달아두지 않으면 언제든지 동작이 변경될 가능성
- 함수가 지금 당장 null을 리턴하지 않아도, 미래에는 변경될 수도 있다는 것을 염두해 둬야 함

### **플랫폼 타입 사용의 문제점**

- 코틀린에서도 플랫폼 코드를 사용할 수 있으나 플랫폼 타입은 안전하지 않으므로 제거하는 것이 좋음

```java
public class JavaClass {
    public String getValue() {
        return null;
    }
}
```

```kotlin
fun statedType() {
    val value: String = JavaClass().value
    / ...
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
	  // ...
    println(value.length)
}
```

- 위 statedType, platformType 모두 NPE가 발생
- `statedType`은, 자바에서 가져오는 값을 가져오는 위치에서 NPE 발생
    
    ⇒ null이 아니라고 예상했지만, null이 나오는 것을 쉽게 알 수 있고, 쉽게 수정 가능
    
- `platformType`은, 값을 활용할때 NPE 발생
    
    ⇒ 플랫폼 타입으로 지정된 변수는 nullable일 수도 있고, 아닐 수도 있음
    
    ⇒ 실제로 활용하는 라인에서 발생하고, 오류를 찾는데 오랜 시간이 걸림
    

```kotlin
interface UserRepo {
    fun getUserName() = JavaClass().value
}

class RepoImpl: UserRepo {
    override fun getUserName(): String? {
        return null
    }
}

fun main() {
    val repo: UserRepo = RepoImpl()
    val text: String = repo.getUserName() // 런타임 때 NPE
    println("User name length is ${text.length}")
}
```

- 인터페이스 메서드의 inferred 타입이 플랫폼 타입이므로, 누구나 nullable 여부를 지정할 수 있는데, 사용하는 쪽에서 nullable이 아니라고 받아들였다면 문제가 발생
    
    ⇒ 플랫폼 타입이 전파(다른 곳에서 사용)되는 일은 굉장히 위험하고 제거하는 것이 좋음
    

---

# 정리

- 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 함
- 플랫폼 타입은 사용하는 코드 말고도, 이를 활용하는 곳까지 영향을 줄 수 있으므로 제거하는 것이 좋음
- 또한 연결된 자바 코드에서 nullable 여부를 지정하는 어노테이션을 활용하는 것이 좋음

---