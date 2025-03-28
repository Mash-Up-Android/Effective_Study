# 가변성을 제한하라

> 코틀린은 정말 안전한 언어다.
이 언어를 정말로 안전하게 사용해 보자.

즉, **오류가 덜 발생하는 코드를 작성**해 보자!


읽고 쓸 수 있는 프로퍼티인 `var`, `mutable` 객체 등의 요소로 **상태**를 가질 수 있다는 것은..

> ⚠
요소가 상태를 갖는 것은 양날의 검이다 라는 뜻

- 프로그램 이해와 디버그가 어렵다
    - 상태를 갖는 부분들의 관계를 이해해야 한다
    - 추적하기도, 코드를 수정하기도 어렵다
- 가변성은 코드 추론하기에 어렵다
    - 시점에 따라 값이 변하므로 추론하기 어렵다
- 멀티스레드일 경우, 동기화가 필요하다
- 테스트가 어렵다
    - 모든 상태를 테스트 해야 한다


아직은.. 이론적이라 잘 와닿지 않았다

코드로 알아보자.

```kotlin
import kotlin.concurrent.thread

fun main() {
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            num += 1
        }
    }

    Thread.sleep(5000)
    println(num)
}
```

위 코드는 멀티스레드를 활용해 num += 1 연산을 하는 코드다

이때, 일부 연산이 이뤄지지 않아서 num에 있는 값은 1000이 아닌 다른 값들이 랜덤하게 나오게 된다

### 왜 1000이 나오지 않을까? 경쟁 조건에 의해 이러한 문제가 발생한다

num += 1은 세 가지 연산으로 이뤄진다.

1. num 값을 읽음
2. 읽은 값에 1을 더함
3. 더한 값을 num에 저장함

멀티 스레드 환경에서 여러 스레드가 이 연산을 동시에 수행한다면 어떨까?

서로 읽기/쓰기를 동시에 하기 때문에 연산이 겹쳐, 예상치 못한 값이 저장되는 것이다.

> num이 0이라는 상태라고 하면,
스레드 A가 num을 읽고 1을 저장하기 전
스레드 B도 num을 읽고 1을 더한 후 저장할 수 있다

A, B는 num의 값이 0으로 동일한 상태에서 각자 1을 더하고 저장하면..

num이 2가 증가해야 하지만, 실제로 num은 1만 증가한 것이 된다


## 동기화를 해본다면?

동기화란, 멀티스레드 환경에서 여러 스레드가 공유 자원에 동시에 접근하지 못하게 하는 것이다

```kotlin
import kotlin.concurrent.thread

fun main() {
    val lock = Any()
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            synchronized(lock) {
                num += 1
            }
        }
    }
    Thread.sleep(1000)
    println(num)
}
```

`synchronized(lock)`  블록 내에서 실행되는 코드는 **하나의 스레드**만 실행할 수 있다.

즉, 여러 스레드가 동시에 이 블록에 진입하는 것을 막아주어 상호 배제를 보장하는 것이다

1. lock이라는 임의의 객체를 사용하여 동기화 된 블록을 설정한다
2. 다른 스레드가 synchronized(lock) 블록에 진입하려면?
    1. lock 객체가 상호 배타적 잠금 역할을 한다
    2. 먼저 블록을 실행 중인 스레드가 완료 될 때까지 기다려야 한다
    3. 현재 스레드가 완료되면 다음 스레드가 접근한다
3. 모든 스레드는 연산을 순차적으로 실행한다

→ 경쟁 조건이 발생하지 않는다!

# 코틀린에서 가변성을 제한하기

코틀린은 불변 객체를 만드는 등 가변성을 제한하는 것이 매우 쉽다.

가장 많이 사용되는 것들을 알아보자

## 1. 읽기 전용 프로퍼티(val)을 사용한다

```kotlin
val a = 10
a = 20 // 변경 못해!
```

읽기 전용 프로퍼티 val을 사용하면 a는 마치 값처럼 쓰인다.

일반적으로는 값이 변하지 않기 때문이다

하웨버, 완전히 변경 불가능 하지는 않다

val 객체가 mutable이라면, 내부적으로는 변경이 된다

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

println(list) // [1, 2, 3, 4]
```

뭐야 가변성을 제한한다면서 왜 변해! 라고 생각했다.

“**읽기 전용**” 이라는 말을 생각해 보자.

list = mutableListOf(4,5,6) 처럼, `재할당하는 것이 불가능` 할 뿐이다.

값이 변할 수 없는 것과 프로퍼티를 읽을 수만 있는 속성을 구분해서 생각하자.

| **개념** | **값이 변할 수 없는 것 (Immutable)** | **읽기 속성 (Read-only Property)** |
| --- | --- | --- |
| **내부 상태 변경 가능 여부** | 내부 상태 변경 불가능 | 내부 상태 변경 가능 (mutable 객체일 경우) |
| **재할당 가능 여부** | 재할당 가능 | 재할당 불가능 |

### 사용자 정의 게터로도 정의할 수 있다

```kotlin
var a = "아린"
var b = "김"
val fullName
    get() = "$a $b"

fun main() {
    println(fullName) // 아린 김
    a = "태희"
    println(fullName) // 태희 김
}
```

var 프로퍼티를 사용하는 val 프로퍼티는 var 프로퍼티로 변할 수 있다

val 프로퍼티는 getter를 사용하여, 동적으로 값을 계산한다

“읽는 시점” 값을 반환하므로, 프로퍼티 값을 저장하지 않고 매번 계산해서 제공할 수 있다

| var | val |
| --- | --- |
| getter 제공 | getter만 제공 |
| setter 제공 |  |

```kotlin
package 이펙티브코틀린

interface Element {
    val active: Boolean
}

class ActualElement : Element {
    override var active: Boolean = false
}
```

이처럼 val 값을 var로 오버라이드 할 수도 있다!

>💡
val은 변경될 수는 있지만, 재할당 할 수는 없으므로 동기화 문제를 줄일 수 있다
> 
var보다는 val을 사용하는 습관을 들이자.


### 완전히 변경할 필요가 없다면 final 프로퍼티를 사용해서 코드 예측성을 높이자

```kotlin
package 이펙티브코틀린

val a: String? = "아린"
val b: String? = "김"

val fullName
    get() = a?.let { "$it $b" }

val fullName2 = a?.let { "$it $b" }

fun main() {
    if (fullName != null) {
        println(fullName.length)    // 스마트 캐스트 불가
    }

    if (fullName2 != null) {
        println(fullName2.length)
    }
}
```

fullName은 값을 사용하는 시점에서 읽으므로, 스마트 캐스트가 불가하다

fullName2는 final이고, 게터를 갖지 않으므로 스마트 캐스트가 가능하다

## 2. 가변 컬렉션과 읽기 전용 컬렉션을 구분한다

코틀린의 컬렉션 인터페이스 계층은 **읽기 전용 컬렉션**과 **변경 가능한 컬렉션**으로 나뉜다

| **읽기 전용 인터페이스** | **변경 가능한 인터페이스** | **설명** |
| --- | --- | --- |
| **`Iterable<T>`** | **`MutableIterable<T>`** | 순회(iteration)를 제공하는 최상위 인터페이스 |
| **`Collection<T>`** | **`MutableCollection<T>`** | 요소 개수 확인, 포함 여부 확인 등의 기능을 제공 |
| **`List<T>`** | **`MutableList<T>`** | 순서가 있는 컬렉션, 인덱스를 기반으로 접근 가능 |
| **`Set<T>`** | **`MutableSet<T>`** | 중복을 허용하지 않는 컬렉션 |

mutable이 붙은 인터페이스는

대응되는 읽기 전용 인터페이스를 상속받아서

**변경을 위한 메서드를 추가**한 것!

쉽다. 고 생각했지만

```kotlin
package 이펙티브코틀린

inline fun <T, R> Iterable<T>.map(transformation: (T) -> R): List<R> {
    val list = ArrayList<R>()
    for (elem in this) {
        list.add(transformation(elem))
    }
    return list
}
```

읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 의미는 아니지만, 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없다?

어렵다.

map이라는 확장 함수처럼, 새로운 컬렉션을 반환할 수는 있지만

컬렉션 자체 요소의 수정은 불가능 하다는 말

순회는 할 수 있지만, 새로운 컬렉션을 만들거나 다른 형태로 변환하는 건 가능.

더 쉬운 예시를 보자.

```kotlin
val numbers = listOf(1, 2, 3)
val squaredNumbers = numbers.map { it * it }
println(squaredNumbers)  // [1, 4, 9]
```

listOf는 불변이지만, map 함수를 사용해 새로운 리스트를 생성했다.

numbers 자체의 값은 변하지 않았다.


>읽기 전용 컬렉션은 내부 데이터를 수정하는 메서드가 없어, 원본을 수정하는 것이 불가하다
> 
>대신, 새로운 컬렉션을 반환하는 메서드들을 활용하자. 예를 들어 map!

이를 통해, 내부적으로 불변하지 않은 컬렉션이어도

외부적으로는 불변한 것처럼 보이게 함으로써, 안정성을 높일 수 있는 것이다

### 하지만 컬렉션 다운캐스팅을 한다면? (허용해서는 안되지만.)

다운캐스팅이란, 부모 타입을 자식 타입으로 변환하는 작업이다

```kotlin
val list = listOf(1,2,3)

// 이거 하지 마라.
if (list is MutableList) {
		list.add(4) // 다운캐스팅 후 요소를 추가하는 것. 하지마!
}
```

컬렉션을 다운캐스팅하여 읽기 전용 컬렉션을 `MutableList` 변환하는 것은 **코틀린의 추상화 원칙을 위반**하며, 예측 불가능한 결과를 초래한다

또한, 플랫폼에 따라 결과가 다르다

JVM에서는 자바의 Arrays.ArrayList를 반환하나,

코틀린과 같은 다른 플랫폼에서는 다른 구현체를 반환할 수도 있다.

코틀린은 읽기 전용 컬렉션과, 변경 가능한 컬렉션을 명확히 구분해 설계했다

다운캐스팅은 이 계약을 깨고, 내부에 의존하게 만든다.

→ 코드의 유연성과 확장성을 저해하고, 유지보수가 어려워진다

### 그렇다면 안전하게 복제를 하자

```kotlin
package 이펙티브코틀린

fun main() {
    val list = listOf(1, 2, 3)
    val mutableList = list.toMutableList()
    mutableList.add(4)
}
```

**`toMutableList()`** 함수를 활용해 읽기 전용을 복제해서, 새로운 mutable 컬렉션을 만든다.

기존의 객체는 여전히 불변하므로, 안전하다!

## 3. 데이터 클래스의 copy

## Immutable 객체의 장점!

1. 한 번 정의된 상태를 유지하므로 코드 이해가 쉽다
2. 충돌이 이뤄지지 않으므로 병렬 처리를 안전하게 할 수 있다
3. 이 객체에 대한 참조는 변경되지 않아서 쉽게 캐시할 수 있다
4. 방어적 복사본을 만들 필요가 없다
    1. 방어적 복사란 객체를 외부에 노출할 때 원본 객체가 변경되지 않도록 복사본을 만들어 반환하는 것이다
    2. 의도치 않은 수정을 방지하기 위함
    3. 방어적 복사는 다음과 같다

    ```kotlin
    class MutablePerson(private val name: String, private val birthDate: Date) {
        fun getBirthDate(): Date {
            return Date(birthDate.time) // 방어적 복사 수행
        }
    }
    
    fun main() {
        val person = MutablePerson("Alice", Date())
        val birthDate = person.getBirthDate()
    
        // 외부에서 birthDate를 변경
        birthDate.time = 0 // 원본 객체는 보호됨
    }
    ```

   불변 객체로 간단하게 작성할 수 있다

    ```kotlin
    data class ImmutablePerson(val name: String, val birthDate: Date)
    
    fun main() {
        val person = ImmutablePerson("Alice", Date())
        val birthDate = person.birthDate
    
        // 외부에서 변경 시도 (불가능)
        // birthDate.time = 0 // 컴파일 에러 발생
    }
    
    ```

5. 객체 복사시 깊은 복사를 하지 않아도 된다
    1. 깊은 복사란 중첩된 모든 참조 객체까지 새로운 메모리에 복사하는 기법이다
    2. 기존 객체와는 완전히 독립적인 새로운 객체를 생성하기 위함
    3. 얕은 복사는 다음과 같다

        ```kotlin
        data class MutablePerson(val name: String, val friends: MutableList<String>)
        
        fun main() {
            val person1 = MutablePerson("Alice", mutableListOf("Bob", "Charlie"))
            val person2 = person1.copy()
        
            person2.friends.add("Dave") // person1.friends도 영향을 받음
            println(person1.friends) // [Bob, Charlie, Dave]
        }
        ```

       중첩된 객체 상태를 변경하면 원본 데이터도 영향을 받는다

    4. 불변 객체는 상태 변경이 불가하여 참조만 복사해도 안전함. 깊은 복사를 수행할 필요가 없다

    ```kotlin
    data class ImmutablePerson(val name: String, val friends: List<String>)
    
    fun main() {
        val person1 = ImmutablePerson("Alice", listOf("Bob", "Charlie"))
        val person2 = person1.copy()
    
        // 친구 목록을 수정하려고 하면 에러 발생
        // person2.friends.add("Dave") // 컴파일 에러
    }
    ```

6. 다른 객체를 만들 때 사용하기 좋다
7. 실행을 더 쉽게 예측할 수 있다
8. **Mutable과 달리 Set, Map의 key로 사용할 수 있다**


>mutable은 예측하기 어렵고 위험하다.
>
>immutable은 객체를 변경할 수 없다.
>
>즉, immutable 객체는 자신의 일부를 수정한 새로운 객체를 만들어 내는 메서드를 가져야 한다

```kotlin
package 이펙티브코틀린

fun main() {
    class User(
        val a: String,
        val b: String
    ) {
        fun withB(b: String) = User(a, b)
    }

    var user = User("아린", "김")
    user = user.withB("연아")
    println(user)
}
```

이렇게 withB 메서드를 추가해서 새로운 객체를 만들어 준다.

하지만 이렇게 함수를 하나하나 만드는 건 너무 귀찮으니, data 한정자를 사용하는 것이다

```kotlin
package 이펙티브코틀린

fun main() {
    data class User(
        val a: String,
        val b: String
    )

    var user = User("아린", "김")
    user = user.copy(a ="연아")
    println(user)
}
```

data 한정자는 `copy`라는 메서드를 만들어준다

copy는 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들 수 있게 해준다

> 즉, 코틀린에서는 이처럼 불변 특성을 가지는 데이터 모델 클래스를 만들 수 있다
모델 클래스로 불변하게 만드는 습관을 들이자.
>

# 다른 종류의 변경 가능 지점

다음 두 코드는 변경 가능 지점이 어떻게 다른지 살펴보자.

하나는 mutable 컬렉션, 다른 하나는 var 프로퍼티다.

```kotlin
package 이펙티브코틀린

fun main() {
    val list1 : MutableList<Int> = mutableListOf()
    var list2 : List<Int> = listOf()

    println(list1)
    println(list2)
    
    list1.add(1) // list1.plusAssign(1)
    list2 = list2 + 1 // list2.plus(2)

    println(list1)
    println(list2)
    
    list1 += 1
    list2 += 1

    println(list1)
    println(list2)
}
```

- **`list1`**: **`MutableList`** 타입으로 선언되어, 요소 추가/삭제 가능하지만 **`val`**로 선언되었으므로 참조 자체는 변경할 수 없음
    - list1.add(1) 연산은 실제로 컬렉션 자체를 변경하는 것
    - 기존 컬렉션에 항목을 추가/삭제 하는 것이다
    - 기존 객체를 수정하므로 메모리 사용량이 적다
- **`list2`**: **`List`** 타입으로 선언되어 읽기 전용 컬렉션이지만 **`var`**로 선언되었으므로 새로운 리스트로 재할당이 가능함
    - list2 = list2 + 1 은 list2에 새로운 요소 1을 추가한 List를 할당하는 연산임
    - 중요한 점은 원본 lsit2는 변경되지 않고, 새로운 List 객체를 할당한다는 것임
    - 기존 리스트를 변경하는 것이 아닌, 새로운 리스트를 재할당
    - 매번 새로운 객체를 생성하여 메모리 사용량이 증가한다

| **plusAssign** | 기존 객체 내부 상태를 변경
항상 Unit이며, 값을 반환하지 않음 

| **plus** | 새로운 객체 반환 |

### 두 가지의 변경 가능 지점 위치가 다르다

첫 번째는 구체적인 리스트 구현 **내부**가,

두 번째는 프로퍼티 **자체**가 변경 지점이다

프로퍼티 자체를 변경하는 것이 멀티스레드 처리의 안정성에 더 좋다.

### mutable 프로퍼티에 읽기 전용 컬렉션을 사용하자

```kotlin
var announcements = listOf<Announcement>()
		private set
```

var 라는 mutable 프로퍼티, listOf라는 읽기 전용 컬렉션

내부 코드에서는 필요에 따라 새로운 컬렉션으로 프로퍼티를 재할당할 수 있다.

외부로는 불변성, 내부적으로는 상태 변경을 허용한다 → **캡슐화와 안정성을 동시에 확보**

외부에서는 안전하게 데이터를 조회

내부에서는 필요한 변경 작업이 자유로워 안정적임

### 이건 자제하자

```kotlin
var list = mutableListOf<Int>()
```

→ 프로퍼티, 컬렉션이 모두 변경 가능해서 두 지점 모두에 대해 동기화를 구현해야 함

## 변경 가능 지점을 노출하면 안된다

mutable 객체를 외부에 노출하는 것은 굉장히 위험하다는 것을 알아보자

```kotlin
package 이펙티브코틀린

data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}

fun main() {
    val userRepository = UserRepository()

    val storedUsers = userRepository.loadAll()
    storedUsers[4] = "아린"

    println(userRepository.loadAll())
}
```

UserRepository 내부의 storedUsers는 `private`이므로 직접 접근할 수 없지만

위 코드에서 loadAll() 함수는 storedUsers의 참조, 변경 가능 지점을 그대로 반환하고 이 값을 변경하면 원본 데이터도 바뀐다

**캡슐화가 깨진다.**

### 해결 1. 방어적 복제: 리턴되는 mutable 객체를 복제하자

data 한정자로 만들어지는 copy 메서드를 활용하자.

```kotlin
private val user : MutableUser()

fun get(): MutableUser {
		return user.copy()
}
```

### 해결 2. 업캐스트로 가변성을 제한하자

```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): Map<Int, String> {
        return storedUsers
    }
}

```

읽기 전용인 Map 타입으로 업캐스트 했다.

가변성 제한!