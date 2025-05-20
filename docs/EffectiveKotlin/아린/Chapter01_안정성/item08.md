# 적절하게 null을 처리하라


간단히 null을 리턴하는 사례를 보자

- **`String.toIntOnNull()`**
    - String을 Int로 적절하게 변환할 수 없을 경우 null 리턴
- **`Iterable<T>.firstOrNull() → Boolean`**
    - 주어진 조건에 맞는 요소가 없을 경우 null 리턴

nullable 값을 처리할 때 최대한 명확한 의미를 갖도록 하자

nullable 타입은 다음의 세 가지 방법으로 처리한다

## 첫 번째 방법 : `?.`, `스마트 캐스팅`, `Elvis` 연산자

```kotlin
printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅

// Elvis 연산자
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("")
```

애플리케이션 사용자 관점에서 가장 안전한 방법이다

개발자 입장에서도 편리하여 사실상 가장 많이 사용한다

또한 Elvis 연산자는 오른쪽에 `return`, `throw` 등의 모든 표현식이 허용된다

컬렉션 처리시, 빈 값을 나타낼 때에는 null 대신 빈 컬렉션을 사용해야 한다

> Collection<T>?.orEmpty()
→ null이 아닌 List<T>를 리턴받는다
>

### 방어적 프로그래밍

모든 가능성을 올바른 방식으로 처리하기

예를 들어, null일 때 출력하지 않는 것

상황을 처리할 수 있는 올바른 방법으로 안정성을 업그레이드 한다

### 공격적 프로그래밍

근데 사실 방어적 프로그래밍처럼 모든 상황을 안전하게 처리하긴 어렵다

예상치 못한 상황 발생시, 개발자에게 문제를 알려서 수정하게 만드는 것이 공격적 프로그래밍이다

`require`, `check`, `assert` 등이 이에 활용되는 도구다

> 방어적 프로그래밍과 공격적 프로그래밍은 안정성을 위해 모두 필요하다
이 둘을 이해하고 적절하게 사용할 수 있도록 하자
>

## 두 번째 방법 : 오류 `throw`

개발자가 오류를 찾기 쉽도록 오류를 강제로 발생하게 하는 방법이다

이러한 상황에서는 throw, requireNotNull, checkNotNull 등을 활용한다

```kotlin
fun process(user: User) {
		requireNotNull(user.name)
		val context = checkNotNull(context)
		val networkService = 
				getNetworkService(context) ?:
				throw NoInternetConnection()
}
```

### *not-null assertion(!!) 사용은 Nope*

- 코드가 짧아서 남용하기 쉬움
- 어떤 설명도 없는 제네릭 예외 NPE가 발생함. 예외는 예상치못한 잘못된 부분을 알리기 위해 발생하는 것이므로, 제네릭 예외보단 명시적 오류를 사용해 더 많은 정보를 제공할 수 있어야 함
- !!은 nullable 타입이 확실하게 null이 나오지 않는 상황에서 쓰여지지만, 현재 확실하다고 해서 미래에도 그렇다는 건 아니기 때문에 남용하지 않아야함

> `!!` 연산자를 본다면 반드시 의심하고. 이상하게 생각하도록 하자.
>

## 방법 3 : nullable 타입이 나오지 않도록 함수나 프로퍼티를 리팩토링

### 의미없는 nullable을 피하자

nullable은 어떻게든 처리해야 해서 추가 비용이 발생한다

따라서 꼭 필요한 경우가 아니라면 nullable 자체를 피하자

이유없는 null을 사용한다면 다른 개발자 입장에서 `!!` 연산자를 사용할 수도 있고, 코드를 더럽히는 예외처리가 추가적으로 필요하게 된다

nullable을 피할 수 있는 방법은 다음과 같다

- List<T>의 `get`, `getOrNull`와 같은 함수 사용
- 클래스 생성 이후 확실하게 설정되는 값에는 `lateinit`, `notNull` 델리게이트 사용
- null 대신 빈 컬렉션 리턴
    - List<Int>? 와 같은 컬렉션은 빈 컬렉션, null의 의미가 완전히 다르다
      null은 컬렉션 자체가 없음을 의미하고, 빈 컬렉션은 컬렉션에 요소가 없다는 것을 뜻한다
- nullable enum 대신 None enum 사용

### lateinit 프로퍼티

**프로퍼티를 처음 사용하기 전에 반드시 초기화될 상황**에 사용한다

예를 들어,

@BeforeEach와 같이 다른 함수들 보다 먼저 호출되는 함수에서 프로퍼티를 설정할 경우

Activity의 onCreate에서 선언할 경우

물론, lateinit에도 비용이 발생한다

초기화 전에 값을 사용하려고 하면 예외가 발생하겠지만, 오히려 좋다

예외가 발생되어 그 값을 수정할 수 있기 때문이다

lateinit이 nullable과 다른점

- !! 연산자로 언팩 하지 않아도 된다
- 이후 어떤 의미를 나타내는 용도로 null을 사용할 때 nullable로 만들 수 있다
- 프로퍼티 초기화 이후엔 초기화 전의 상태로 돌아갈 수 없다

```kotlin
class UserControllerTest {
		private lateinit var dao: UserDao
		
		@BeforeEach
		fun init() {
				dao = mockk()
				controller = UserController(dao)
		}
}
```

### notNull 델리게이트

반대로 Int, Long, Double, Boolean 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화 할 경우엔 lateinit을 사용할 수 없다

이 경우에는 lateinit보단 조금 느리지만 notNull 델리게이트를 쓴다

```kotlin
private var doctorId: Int by Delegates.notNull()
```