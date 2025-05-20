# 최대한 플랫폼 타입을 사용하지 말라

> **플랫폼 타입?**
다른 언어에서 전달되어 nullable 인지 확인할 수 없는 타입
****코틀린에서 자바와 같은 다른 언어의 타입을 다룰 때 발생하는 특수한 타입
>

코틀린의 주요 기능 중 하나인 **null-safefy**

하지만, 자바에서는 모든 것이 nullable 일 수 있다

자바에서 온 코드를 최대한 안전하기 접근하기 위해 코틀린에서는 어떻게 해야 할까?

1. 이 플랫폼 타입을 nullable으로 가정한다.
2. null을 리턴하지 않는 것이 확실하다면 `!!` 연산자 활용
    1. `!!` 활용은 왠지 찜찜한데.

   `!!` 연산자는 무조건 null이 아니라고 가정하는 위험한 연산자

   이걸 남발할 경우 NPE가 발생할 가능성이 높아지므로 가급적 사용하지 않는 게 좋다

   👉 대신, `nullable(?`), `requireNotNull`, `elvis(?:)` ****연산자 사용을 고려하자!


### 자바에서 List<User>를 리턴한다면?

어노테이션이 따로 달려있지 않는 한, 코틀린은 이 리스트의 내부와 리스트 자체 둘 다 null check 해줘야 한다

심지어 List<List<User>>를 반환한다고 하면 훨씬 복잡하다

리스트는 map이나 fliterNotNull 등 메서드를 제공한다 쳐도,

다른 제네릭 타입에서는 널 체크를 하는 것 자체가 코드의 복잡함을 증가시키는 것

```kotlin
package 이펙티브코틀린

fun main() {
    // 자바
    public class UserRepo {
        public User getUser() {
        
        }
    }
    
    // kotlin
    val repo = UserRepo()
    val user1 = repo.user   // User!
    val user2: User = repo.user // User
    val user3: User? = repo.user    // User?
    
    val users: List<User> = UserRepo().users
    val users: List<List<User>> = UserRepo().groupedUsers
}
```

위의 코드처럼 플랫폼 타입은 타입 이름 뒤에 `!` 기호를 붙여 표기한다

어노테이션이 직접 나타나진 않지만 선택적으로 사용하는 것.

### 자바 코드에서는 함수가 당장 null을 리턴하지 않더라도 미래에는 변경될 수도 있다는 점을 염두해두자

자바를 코틀린과 함께 사용한다면, 플랫폼 타입을 사용할 때에는 항상 null에 대한 위험에 주의를 기울여야 한다

- 어노테이션을 명시적으로 표기하거나 주석 달기
    1. @Nullable
    2. @NotNull

실제로, 안드로이드 API가 코틀린 친화적이라고 불리는 이유가 이러한 어노테이션들이 붙어있기 때문

> ex)
JetBrains
Android
…
>

### 코틀린에서 본다면 플랫폼 타입은 최대한 빠르게 제거하는 것이 낫다

아래의 `statedType()`과 `platformType()`은 둘다 Null Pointer Exception이 발생하는데

어떤 차이가 있는지 보자

```kotlin
package 이펙티브코틀린

fun main() {
    // 자바
    public class JavaClass {
        public String getValue()
        {
            return null;
        }
    }

    // kotlin
    fun statedType() {
        val vaue: String = JavaClass().value  // NPE
        println(value.length)
    }
    
    fun platformType() {
        val value = JavaClass().value
        println(value.length)  // NPE
    }
}
```

`statedType()`  함수는 **자바에서 값을 가져오는 위치**에서 NPE 발생

→ null이 리턴된다는 것을 굉장히 쉽게 알 수 있어 코드 수정에 용이

`platformType()` 함수는 **값을 활용할 때** NPE 발생

→ 이 변수를 한 두번 안전하게 사용해도 다른 사용자가 잘못 사용할 가능성이 높음

→ 오류를 찾는데 오랜 시간을 소요하게 됨

이처럼 플랫폼 타입은 많은 위험성을 초래한다.

```kotlin
val value: String? = JavaClass().value // nullable로 안전하게 처리하자
```

추가로 밑의 인터페이스 사용 사례를 보자

```kotlin
package 이펙티브코틀린

interface UserRepo {
    fun getUserName() = JavaClass().value  // 메서드의 추론 타입이 플랫폼 타입이다.
}

class RepoImpl : UserRepo {
    override fun getUserName(): String? {
        return null
    }
}

fun main() {
    val repo: UserRepo = RepoImpl()
    val text: String = repo.getUserName()   // 런타임 시 NPE
    println(text.length)
}
```



플랫폼 타입은 컴파일 타임에 null 여부를 검증하지 않으므로, 런타임에서 NPE가 발생할 가능성이 높다

안전하게 플랫폼 타입은 쓰지 말자