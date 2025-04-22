# 8. 적절하게 null을 처리하라

---

---

# 개요

null은 ‘값이 부족하다(lack of value)’는 것을 나타냄

프로퍼티가 null이라는 것은 값이 설정되지 않았거나, 제거되었다는 것을 나타냄

함수가 null을 리턴하는 것은 함수에 따라 여러 의미를 가질 수 있음

- `String.toIntOrNull()`은 String을 Int로 변환할 수 없는 경우 null 리턴
- `Iterable<T>.firstOfNull(() → Boolean`은 주어진 조건에 맞는 요소가 없을 경우 null 리턴

이처럼 null은 최대한 명확한 의미를 갖는 것이 좋음 ⇒ nullable 값을 처리해야하기 때문

```kotlin
val printer: Printer? = getPrinter()
printer.print() // 컴파일 오류

printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅
printer!!.print()   // not-null assertion
```

nullable 타입을 처리하는 3가지 방법

- ?., 스마트 캐스팅, Elvis 연산자 등을 활용해서 null을 안전하게 처리
- 오류를 throw
- 함수 또는 프로퍼티를 리펙토링해서 nullable 타입을 나오지 않게 바꿈

---

# null을 안전하게 처리하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법인 safe call과 스마트 캐스팅이 있음

```kotlin
printer?.print() // safe call
if (printer != null) printer.print() // 스마트 캐스팅
```

- 스마트 캐스팅이 작동하지 않는 경우
    
    ### ✅ 스마트 캐스팅이 **동작하지 않는 경우**
    
    ### 1. **`val`이 아닌 `var`인 경우**
    
    ```kotlin
    var printer: Printer? = getPrinter()
    
    if (printer != null) {
        printer.print() // ❌ 컴파일 오류: 스마트 캐스팅 안됨
    }
    ```
    
    - `var`은 값이 **중간에 바뀔 수 있어서**, 컴파일러는 `if` 이후에도 `null`이 아닐 거라고 **보장할 수 없음**.
    - 해결: `val`로 바꾸면 가능함
    
    ```kotlin
    val printer: Printer? = getPrinter()
    
    if (printer != null) {
        printer.print() // ✅ 스마트 캐스팅 동작
    }
    ```
    
    ---
    
    ### 2. **`val`이라도 커스텀 getter가 있는 경우**
    
    ```kotlin
    val printer: Printer?
        get() = fetchPrinter() // getter가 호출될 때마다 값이 바뀔 수 있음
    
    if (printer != null) {
        printer.print() // ❌ 스마트 캐스팅 안됨
    }
    ```
    
    - 이유: 컴파일러는 `printer` 호출할 때마다 **다른 값이 반환될 수도 있다고 판단**함
    
    ---
    
    ### 3. **람다나 클래스 안에서 외부 변수 참조 시**
    
    ```kotlin
    fun printIfNotNull(printer: Printer?) {
        if (printer != null) {
            val run = Runnable {
                printer.print() // ❌ 스마트 캐스팅 안됨
            }
            run.run()
        }
    }
    ```
    
    - 람다 내부에서는 **외부의 `printer`가 언제 바뀔지 알 수 없기 때문에**, 스마트 캐스팅이 안 됨
    
    ---
    
    ### 4. **다중 스레드 상황**
    
    ```kotlin
    @Volatile var printer: Printer? = null
    
    if (printer != null) {
        printer.print() // ❌ 스마트 캐스팅 안됨 (다른 스레드에서 null로 바꿀 수 있음)
    }
    ```
    
    - `@Volatile`이 붙은 변수는 **스레드 간 변경 가능성**이 있어서, 컴파일러가 스마트 캐스팅을 하지 않음
    
    ---
    
    ### ✅ 정리
    
    | 조건 | 스마트 캐스팅 동작 여부 |
    | --- | --- |
    | `val` 변수 | ✔️ (getter가 없으면) |
    | `var` 변수 | ❌ |
    | 사용자 정의 getter | ❌ |
    | 람다나 클래스 내부에서 사용 | ❌ |
    | 멀티스레드 환경 (`@Volatile`) | ❌ |
    
    ---
    

두 가지 모두 printer가 null이 아닐 때 `print()` 함수를 호출

앱 사용자 관점에서 가장 안전한 방법

개발자에게도 편리하여, nullable 값을 처리할 때 이 방법을 가장 많이 활용

코틀린은 nullable 변수와 관련된 처리를 광범위하게 지원

대표적으로 인기 있는 다른 방법은 Elvis 연산자를 사용하는 것

Elvis 연산자는 오른쪽에 return 또는 throw를 포함한 모든 표현식이 허용

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

많은 객체가 nullable 관련된 처리를 지원

예를 들어 컬렉션 처리를 할 때 무언가 없다는 것을 나타낼 때는 null이 아닌 빈 컬렉션을 사용하는게 일반적임

따라서 Collection<T>?.orEmpty() 함수를 사용하면 nullabledl 아닌 List<T>를 리턴받음

스마트 캐스팅은 코틀린 규약 기능(contracts feature)을 지원

> 코틀린의 규약 기능(contracts feature)?
→ 컴파일러에게 "이 함수를 호출하면 어떤 조건이 보장된다"는 논리적인 단서를 제공하는 기능으로, 주로 스마트 캐스팅을 유도하거나, 컴파일러의 흐름 추론 능력을 강화할 때 사용
> 

이 기능을 사용하면 다음 코드처럼 스마트 캐스팅 가능

```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()}")
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    news.forEach { notifyUser(it) }
}
```

---

# 오류 throw하기

위의 코드에서는 printer가 null일 때, 이를 개발자에게 알리지 않음

하지만 printer가 null이 되리라 예상치 못했다면, print 메서드가 호출되지 않아서 오류를 찾기 어렵게 만듦

따라서 다른 개발자가 어떤 코드를 보고 선입견처럼 ‘당연히 그럴 것이다’라고 생각하게 되는 부분이 있고, 그 부분에서 문제가 발생할 경우 개발자에게 오류를 강제로 발생시키는 게 좋음

오류를 강제 발생시킬 때는 throw, !!, requireNotNull, checkNotNull 등을 활용

```kotlin
fun process(user: User) {
    requireNotNull(user.name)
    val context = checkNotNull(context)
    val networkService = getNetworkService(context) ?: throw NoInternetConnection()
    networkService.getData { data, userData -> 
        show(data!!, userData!!) 
    }
}
```

---

# not-null assertion(!!)과 관련된 문제

nullable을 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 것

그런데 !!를 사용하면 자바에서 nullable을 처리할 때 발생할 수 있는 문제가 똑같이 발생

어떤 대상이 null이 아니라고 생각하고 다루면, NPE 발생

!!은 사용하기 쉽지만, 좋은 해결 방법은 아님

예외가 발생할 때, 어떤 설명도 없는 제너릭 예외(generic exception)가 발생함

또한 코드가 짧고 너무 사용하기 쉽다 보니 남용하게 되는 문제도 있음

!!은 타입은 nullable이지만, null이 나오지 않는다는 것이 거의 확실한 상황에서도 많이 사용됨

하지만 현재 확실하다고, 미래에 확실한 것은 아님

간단한 예로 파라미터로 4개의 숫자를 받고, 이중에서 가장 큰 것을 찾는 함수가 있음

모든 파라미터를 리스트에 넣은 뒤에 max 함수를 사용해서 가장 큰 값을 찾게 설계함

컬렉션 내부에 아무것도 없을 경우 null을 리턴하므로, 최종적으로 nullable을 리턴

이 리턴값이 null일 수 없다는 것을 알고 있는 개발자는 다음과 같이 !! 연산자를 사용하게 됨

```kotlin
fun largestOf(a: Int, b: Int, c: Int, d: Int): Int = listOf(a, b, c, d).max()!!
```

누군가 함수를 리팩토링하면서 컬렉션이 null일 수 있다는 걸 놓쳐 !!는 NPE로 이어질 수 있음

```kotlin
fun largestOf(vararg nums: Int): Int = nums.max()!!

largestOf() // NPE
```

nullability(null일 수 있는지)와 관련된 정보는 숨겨져 있으므로 쉽게 놓칠 수 있음(변수와 비슷)

변수를 일단 선언하고, 이후에 사용하기 전에 값을 할당해서 쓰기로 하고, 다음 코드를 작성

이처럼 변수를 null로 설정하고, 이후에 !! 연산자를 사영용하는 방법은 좋은 방법이 아님

```kotlin
class UserControllerTest {
    private var dao: UserDao? = null
    private var controller: UserController? = null
    
    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }
    
    @Test
    fun test() {
        controller!!.doSomeThing()
    }
    
}
```

이렇게 코드를 작성하면, 이후에 프로퍼티를 계속 언팩(unpack)해야 하므로 사용하기 귀찮

또한 해당 프로퍼티가 실제로 이후에 의미 있는 null 값을 가질 가능성 자체를 차단

이런 코드를 작성하는 올바른 방법은 `lateinit` 또는 `Delegates.notNull`을 사용하는 것

!! 연산자를 쓰거나 명시적으로 예외를 발생시키는 형태로 설계하면, 미래의 어느 시점에서 해당 코드가 오류를 발생시킬 수 있다는 걸 염두에 둬야 함

예외는 예상하지 못한 잘못된 부분을 알려주기 위해 발생하는 것

하지만 명시적 오류는 제네릭 NPE보다는 더 많은 정보를 제공해줄 수 있으므로 !! 연산자를 쓰는 것보단 훨씬 좋음

!! 연산자가 의미 있는 경우는 굉장히 드묾

일반적으로 nullability가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 써야 함

코틀린 대상으로 설계된 API를 활용한다면, !! 연산자를 쓰는 걸 이상하게 생각해야 함

일반적으로 !! 사용을 피해야 함

Detekt 같은 정적 분석 도구는 !! 연산자를 사용하면, 아예 오류가 발생하게 설정돼 있음

!! 연산자를 보면 반드시 조심하고, 뭔가 잘못되어 있을 가능성을 생각해야 함

---

# 의미 없는 nullability 피하기

nullability는 어떻게든 적절하게 처리해야 하므로, 추가 비용이 발생함

따라서 필요한 경우가 아니면, nullability 자체를 피하는 것이 좋음

null은 중요한 메시지를 전달하는 데 쓰일 수 있음

따라서 다른 개발자가 보기에 의미가 없을 때는 null을 안 쓰는 게 좋음

만약 이유 없이 null을 사용했다면, 다른 개발자들이 코드를 작성할 때, 위험한 !! 연산자를 사용하게 되고, 의미 없이 코드를 더럽히는 예외 처리를 해야함

다음은 nullability를 피할 때 사용할 수 있는 방법

- 클래스에서 nullability에 따라 여러 함수를 만들어서 제공할 수도 있음
    
    ⇒ 대표적인 예로 List<T>와 `get`, `getOrNull`함수
    
- 어떤 값이 클래스 생성 후 확실하게 설정된단 보장이 있다면, lateinit 프로퍼티와 notNull 델리게이트를 사용
- 빈 컬렉션 대신 null을 리턴하지 말아야 함
    
    ⇒ `List<Int>?`와 `Set<String?>`같은 컬렉션을 빈 컬렉션으로 둘 때와 null로 둘 때는 의미가 다름
    
    null은 컬렉션 자체가 없다는 걸 나타내므로, 요소가 부족하다는 걸 나타내려면 빈 컬렉션을 사용해야 함
    
- nullable enum, None enum 값은 완전히 다른 의미임
    
    ⇒ null enum은 별도로 처리해야 하지만, None enum 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해서 활용할 수 있다는 의미
    

---

# lateinit 프로퍼티와 notNull 델리게이트

클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 드문 일은 아니지만 분명 존재함

이러한 프로퍼티는 사용 전에 반드시 초기화 해서 사용해야 함

예로 JUnit의 `@BeforeEach`처럼 다른 함수들보다도 먼저 호출되는 함수에서 프로퍼티가 설정되는 경우가 있음

```kotlin
class UserControllerTest {
    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }

    @Test
    fun test() {
        controller!!.doSomething()
    }
}
```

프로퍼티를 쓸 때마다 nullable에서 null이 아닌 것으로 타입 변환하는 것은 바람직하지 않음

이런 값은 테스트 전에 설정될 거라는 것이 명확하므로, 의미 없는 코드가 사용된다고 할 수 있음

이런 코드에 대한 바람직한 해결책은 나중에 속성을 초기화할 수 있는, lateinit 한정자를 사용하는 것임

lateinit 한정자는 프로퍼티가 이후에 설정될 것임을 명시함

```kotlin
class UserControllerTest {
    private lateinit var dao: UserDao?
    private lateinit var controller: UserController?

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao)
    }

    @Test
    fun test() {
        controller.doSomething()
    }

}
```

물론 lateinit을 사용할 경우에도 비용이 발생함

만약 초기화 전에 값을 사용하려고 하면 예외가 발생

처음 사용하기 전에 반드시 초기화가 되어 있을 경우에만 lateinit을 붙이는 것임

만약 그런 값이 사용되어 예외가 발생한다면, 그 사실을 알아야 하므로 예외가 발생하는 것은 오히려 좋음

lateinit는 nullable과 비교해서 다음과 같은 차이가 있음

- !! 연산자로 언팩하지 않아도 됨
- 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수도 있음
- 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없음

lateinit은 프로퍼티를 처음 사용하기 전에 반드시 초기화될 거라고 예상되는 상황에 활용

이러한 상황으로는 라이프 사이클(lifecycle)을 갖는 클래스처럼 메서드 호출에 명확한 순서가 있을 경우가 있음

(ex. Android Activity의 onCreate, iOS UIViewController의 viewDidAppear, 리액트 React.Component의 componentDidMount 등)

lateinit을 사용할 수 없는 경우

⇒ JVM에서 Int, Long, Double, Boolean 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야 하는 경우

→ 이 경우 lateinit보다 느리지만 `Deletages.notNull`을 사용

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean by Delegates.notNull()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
    }
}
```

위 코드처럼 onCreate 때 초기화하는 프로퍼티는 지연 초기화하는 형태로 다음처럼 프로퍼티 위임(property delegation)을 사용할 수도 있음

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by arg(DOCTOR_ID_ARG)
    private var fromNotification: Boolean arg(FROM_NOTIFICATION_ARG)
}
```

프로퍼티 위임을 사용하면, nullability로 발생하는 여러 문제를 안전하게 처리할 수 있음

---