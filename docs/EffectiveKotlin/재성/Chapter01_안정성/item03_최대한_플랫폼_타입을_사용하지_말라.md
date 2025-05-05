### 일단 나는 자바경험이 별로 없음 ㅋ 읽을줄만 앎

코틀린의 Null Safety는 자바에 비해 충분한 장점이 될 수 있지만, 자바에서 사용하던 플랫폼 타입을 사용할 경우 안전함을 보장하기 어렵게 된다.

플랫폼 타입은 외부에서 사용될 때 Null인지 Null이 아닌지 명시적으로 판단하기가 어렵기 때문에 안전한 코드를 작성한다면 피하는게 좋다.

코틀린 타입 추론 기능을 활용해서 아래처럼 작성한다고 했을 때도 문제가 발생한다.

```kotlin
interface UserRepo {
    fun getUserName() = JavaClass().value  // 이게 반환하는 타입이 뭔데 ㅇㅅㅇ
}
```

나는 얘를 Nullable로 의도하고 작성했는데 아린이가 NotNull로 받아들이면 문제가 되지 않을까~

### 플랫폼 타입

기존 Java에서는 @Nullable, @NotNull 어노테이션을 통해 해당 타입의 Nullable을 확인할 수 있다

이걸 코틀린에서 사용할 때 선언된 어노테이션을 통해 타입의 Nullable을 알 수 있는데 이런게 선언되어있지 않다면 우리가 확인할 길이 없음

이런 특수한 유형을 플랫폼 타입이라고 함

```kotlin
public class UserRepo {
    public User getUser() { .. }
    public @NotNull User get유저() { .. }
}

val repo = UserRepo()
val user1 = repo.getUser()  // User!
val user2: User = repo.getUser()  // User
val user3: User? = repo.getUser()  // User?
val user4 = repo.get유저()  // User
val user5: User? = repo.get유저()  // User?
```