# 27. 변화로부터 코드를 보호하려면 추상화를 사용하라

---

---

# 개요

함수, 클래스 등의 추상화로 실질적인 코드를 숨기면, 사용자가 세부 사항을 알지 못해도 괜찮다는 장점 있음

사용자는 함수의 입출력만 알면 되므로, 이후에 실질적인 코드를 원하는대로 수정 가능

ex. 정렬 알고리즘을 함수로 추출하면, 사용하는 코드에 아무런 영향도 주지 않고, 함수 성능 최적화 가능

---

# 상수(constant value

리터럴(`123`, `"ABC"` 등)은 아무것도 설명하지 않아서, 코드에서 반복적으로 등장할때 문제가 됨

이러한 리터럴을 상수 프로퍼티로 변경하면 해당 값에 의미있는 이름을 붙일 수 있으며, 상수의 값을 변경해야 할 때 훨씬 쉽게 변경 가능

- 비밀번호 유효성을 검사하는 예시

```kotlin
const val MIN_PASSWORD_LENGTH = 7

fun isPasswordValid(text: String): Boolean { 
    if (text.length < MIN_PASSWORD_LENGTH) return false
    // ...
}
```

---

# 함수

토스트 메세지를 자주 출력해야 하는 상황에 보통 아래와 같은 코드를 사용해서 토스트 메세지를 출력

```kotlin
Toast.makeText(this, message, Toast.LENGTH_LONG).show()
```

이렇게 많이 사용되는 알고리즘은 다음과 같이 간단한 확장 함수를 만들어서 사용 가능

```kotlin
fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG
) { 
    Toast.makeText(this, message, duration).show()
}

// 사용
context.toast(message)

// 액티비티 또는 컨텍스트의 서브클래스에서 사용할 경우
toast(message)
```

이렇게 일반적인 알고리즘을 추출하면, 토스트를 출력하는 코드를 항상 기억해 두지 않아도 괜찮음

또한 이후에 토스트를 출력하는 방법이 변경되어도, 확장 함수 부분만 수정하면 되므로 유지보수성이 향상됨

만약 토스트가 아니라 스낵바라는 다른 형태의 방식으로 출력해야 한다면 스낵바를 출력하는 확장 함수를 만들고 기존의 `Context.toast()`를 `Context.snackbar()`로 한꺼번에 수정하면 됨

```kotlin
fun Context.snackbar(
    message: String,
    length: Int = Toast.LENGTH_LONG
) {
	// ...
}
```

but 이런 해결 방법은 좋지 않음

내부적으로만 사용하더라도, 함수의 이름을 직접 바꾸는 것은 위험할 수 있음

다른 모듈이 이 함수에 의존하고 있다면, 다른 모듈에 큰 문제가 발생

또한 함수의 이름은 한꺼번에 바꾸기 쉽지만, 파라미터는 한꺼번에 바꾸기가 쉽지 않으므로, 메세지의 지속시간을 나타내기 위한 `Toast.LENGTH_LONG`이 계속 사용되고 있다는 문제도 있음

스낵바를 출력하는 행위가 토스트의 필드에 영향을 받는 것은 좋지 않음

다른 한편으로 스낵바의 enum으로 몯느 것을 변경하는 것도 문제를 발생시킴

```kotlin
fun Context.snackbar(
    message: String,
    duration: Int = Snackbar.LENGTH_LONG
) {
	// ...
}
```

메세지의 출력 방법이 바뀔 수 있다는 것을 알고 있다면, 이때부터 중요한 것은 메세지의 출력 방법이 아니라, 사용자에게 메세지를 출력하고 싶다는 의도 자체임

따라서 메세지를 출력하는더 추상적인 방법이 필요함

토스트 출력을 토스트라는 개념과 무관한 `showMessage`라는 높은 레벨의 함수로 옮긴 예시

```kotlin
fun Context.showMessage(
    message: String, 
    duration: MessageLength = MessageLength.Long
) { 
    val toastDuration = when(duration) {
    	SHORT -> Length.LENGTH_SHORT
        LONG -> Length.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

가장 큰 변화는 **이름**임

일부 개발자는 이름 변경은 그냥 레이블을 붙이는 방식의 변화이므로 큰 차이가 없다고 생각하기도 함

but 이러한 관점은 사실 컴파일러의 관점에서만 유효하고, 사람의 관점에서는 이름이 바뀌면 큰 변화임

함수는 추상화를 표현하는 수단이며, 함수 시그니처는 이 함수가 어떤 추상화를 표현하고 있는지 알려줌

⇒ 따라서 의미있는 이름은 굉장히 중요

함수는 매우 단순한 추상화지만, 제한이 많음

예를들어 함수는 상태를 유지하지 않고, 함수 시그니처를 변경하면 프로그램 전체에 큰 영향을 줄 수 있음

---

# 클래스

이전 메세지 출력 예시를 클래스로 추상화한 예시

```kotlin
class MessageDisplay(val context: Context) { 
    fun show(
    	message: String,
        duration: MessageLength = MessageLength.Long
    ) {
    	val toastDuration = when(duration) {
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }

// 사용
val messageDisplay = MesssageDisplay(context)
messageDisplay.show("Message")
```

클래스가 함수보다 더 강력한 이유는 상태를 가질 수 있으며, 많은 함수를 가질 수 있다는 점 때문

위의 코드에서 클래스의 상태인 context는 기본 생성자를 통해 주입(inject)됨

의존성 주입 프레임워크를 사용하면, 클래스 생성을 위임 가능

```kotlin
@Inject lateinit var meesageDisplay: MessageDisplay
```

또한 mock 객체를 활용해서 해당 클래스에 의존하는 다른 클래스의 기능을 테스트 가능

```kotlin
val messageDisplay: MessageDisplay = mock()
```

게다가 메세지를 출력하는 더 다양한 종류의 메서드를 만들 수 있음

```kotlin
messageDisplay.setChristmasMode(true)
```

이처럼 클래스는 훨씬 더 많은 자유를 보장해 줌

but 여전히 한계가 있는데, 예를 들어 클래스가 `final`이라면 해당 클래스 타입 아래에 어떤 구현이 있는지 알 수 있음

`open` 클래스는 서브 클래스를 대신 제공할 수 있어서 `open` 클래스를 활용하면 좀 더 자유를 얻을 수 있음

---

# 인터페이스

코틀린 표준 라이브러리를 읽어보면, 거의 모든 것이 인터페이스로 표현된다는 것을 확인할 수 있음

- `listOf` 함수는 `List`를 리턴 하는데, 여기서 `List`는 인터페이스임(`listOf`는 팩토리 메서드)
- 컬렉션 처리 함수는 `Iterable` 또는 `Collection`의 확장 함수로서, `List`, `Map` 등을 리턴
    
    → 이것들은 모두 인터페이스임
    
- 프로퍼티 위임은 `ReadOnlyProperty` 또는 `ReadWriteProperty` 뒤에 숨겨짐
    
    → 이것들도 모두 인터페이스임
    
    실질적인 클래스는 일반적으로 `private`임(함수 `lazy`는 `Lazy` 인터페이스를 리턴)
    

라이브러리를 만들 때 내부 클래스 가시성은 제한하고, 인터페이스를 통해 이를 노출하는 코드를 많이 사용

이렇게 하면 사용자가 클래스를 직접 사용하지 못하므로, 라이브러리를 만드는 사람은 인터페이스만 유지한다면, 별도의 걱정없이 자신이 원하는 형태로 그 구현을 변경 가능

즉, 인터페이스 뒤에 객체를 숨김으로써 실질적인 구현을 추상화하고, 사용자가 추상화된 것에만 의존하게 만들 수 있음

⇒ 즉, 결합을 줄일 수 있음

코틀린이 클래스가 아니라 인터페이스를 리턴하는 데에는 이외에도 여러 이유가 있음

ex. Kotlin은 멀티 플랫폼 언어이므로, `listOf`가 Kotlin/JVM, Kotlin/JS, Kotlin/Native에 따라서 구현이 다른 리스트를 리턴함

다른 리스트를 사용하는 이유는 최적화 때문 ⇒ 각 플랫폼의 네이티브 리스트를 사용해서 속도를 높이는 것

어떤 플랫폼을 사용해도 List 인터페이스에 맞춰져 있으므로, 차이 없이 사용 가능

이전 메세지 출력 예시에 인터페이스를 도입한 예시

```kotlin
interface MessageDisplay { 
    fun show(
    	message: String,
        duration: MessageLength = LONG
    )
}

class ToastDisplay(val context: Context) : MessageDisplay { 
    override fun show(
    	message: String,
        duration: MessageLength
    ) {
    	val toastDuration = when(duration) {
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}
```

이렇게 구성하면 더 많은 자유를 얻을 수 있음

이러한 클래스는 태블릿에서는 토스트를 출력하고, 스마트폰에서는 스낵바를 출력하게 할 수도 있음

또 다른 장점은 테스트할 때 인터페이스 페이킹(faking)이 클래스 모킹(moking)보다 간단하므로, 별도의 모킹 라이브러리를 사용하지 않아도 됨

```kotlin
val messageDisplay: MessageDisplay = TestMessageDisplay()
```

마지막으로 선언과 사용이 분리되어 있으므로, `ToastDisplay` 등의 실제 클래스를 자유롭게 변경할 수 있음

다만 사용 방법을 변경하려면, `MessageDisplay` 인터페이스를 변경하고, 이를 구현하는 모든 클래스를 변경해야 함

---

# ID 만들기(nextId)

프로젝트에서 고유 ID(Unique ID)를 사용해야 하는 상황을 가정헀을 때, 가장 간단한 방법은 어떤 정수 값을 계속 증가시키면서 이를 ID로 활용하는 것임

```kotlin
var nextId: Int = 0

//사용
val newId = nextId++
```

그런데 이러한 코드가 많이 사용되면, ID가 생성되는 방식을 변경할 때 문제가 발생하기 때문에 위험함

이 방법은 아래와 같은 문제가 있음

- 이 코드의 ID는 무조건 0부터 시작함
- 이 코드는 thread-safe하지 않음

만약 그대로 이 방법을 사용해야 한다면, 일단 이후에 발생할 수 있는 변경으로부터 코드를 보호할 수 있게 함수를 사용하는 것이 좋음

```kotlin
private var nextId: Int = 0
fun getNextId(): Int = nextId++

// 사용
val newId = getNextId()
```

이제 ID 생성 방식의 변경으로부터 보호는 되지만, ID타입 변경 등은 대응하지 못함

미래의 어느 시점에 ID를 문자열로 변경해야 한다면, 현재 타입에 종속적인 연산들도 전부 수정해줘야 함

이를 최대한 방지하려면, 이후에 ID타입을 쉽게 변경할 수 있게 클래스를 사용하는 것이 좋음

```kotlin
data class Id(private val id: Int)

private var nextId: Int = 0
fun getNextId(): Id = Id(nextId++)
```

더 많은 추상화는 더 많은 자유를 주지만, 이를 정의하고, 사용하는, 이해하는 것이 조금 어려워짐

---

# 추상화가 주는 자유

앞서 추상화하는 몇가지 방법들을 구현할 때는 여러 도구를 활용할 수 있음

- 제네릭 타입 파라미터 사용
- 내부 클래스를 추출
- 생성을 제한(ex. 팩토리 함수로만 객체를 생성할 수 있게 만듦)

but 추상화에는 단점도 존재하는데, 추상화는 자유를 주지만 코드를 이해하고 수정하기 어렵게 만듦

---

# 추상화의 문제

어떤 방식으로 추상화를 하려면 코드를 읽는 사람이 해당 개념을 배우고, 잘 이해해야 함

또 다른 방식으로 추상화를 하려면 또 해당 개념을 배우고, 잘 이해해야 함

물론 추상화의 가시성을 제한하거나, 구체적인 작업에서만 추상화를 도입하는 것은 큰 문제가 없음

그래서 큰 프로젝트에서는 잘 모듈화 해야 함

추상화도 비용이 발생하므로, 극단적으로 모든 것을 추상화해서는 안됨

추상화는 거의 무한하게 할 수 있지만, 어느 순간부터 득보다 실이 많아짐

추상화는 많은 것을 숨길 수 있는 테크닉임

생각할 것을 어느 정도 숨겨야 쉬워지는 것도 사실이지만, 너무 많은 것을 숨기면 결과를 이해하는 것 자체가 어려워짐

추상화를 이해하려면, 많은 예제를 살펴보는 것이 좋음

---

# 어떻게 균형을 맞춰야 할까?

모든 추상화는 자유를 주지만, 코드가 어떻게 돌아가는 것인지 이해하기 어렵게 만듦

극단적인것은 언제나 좋지 않으므로, 추상화의 적당한 정도는 다음과 같은 요소들에 따라서 달라질 수 있음

- 팀의 크기
- 팀의 경험
- 프로젝트의 크기
- 특징 세트 (feature set)
- 도메인 지식

따라서 모든 프로젝트에 따라서 균형이 다를 수 있음

적절한 균형을 찾는 것은 거의 감각에 의존해야 하므로, 수백 시간 이상의 경험이 있어야 할 수 있는 일임

그래도 사용할 수 있는 몇 가지 규칙을 정리해 보면 다음과 같음

- 많은 개발자가 참여하는 프로젝트는 이후에 객체 생성과 사용 방법을 변경하기 어렵움
    
    따라서 추상화 방법을 사용하는 것이 좋음. 최대한 모듈과 부분(part)을 분리하는 것이 좋음
    
- 의존성 주입 프레임워크를 사용하면, 생성이 얼마나 복잡한지 신경쓰지 않아도 됨
    
    클래스 등은 한 번만 정의하면 되기 때문
    
- 테스트를 하거나, 다른 애플리케이션을 기반으로 새로운 애플리케이션을 만든다면 추상화를 사용하는 것이 좋음
- 프로젝트가 작고 실험적이라면, 추상화를 하지 않고도 직접 변경해도 괜찮음
    
    문제가 발생했다면 최대한 빨리 직접 변경하면 됨
    

항상 무언가 변화할 수 있다고 생각하는 것이 좋음

---

# 정리

- 추상화는 단순하게 중복성을 제거해서 코드를 구성하기 위한 것이 아님
- 추상화는 코드를 변경해야 할 때 도움이 됨
- 따라서 추상화를 사용하는 것은 굉장히 어렵지만, 이를 배우고 이해해야 함
- 다만 추상적인 구조를 사용하면, 결과를 이해하기 어려움
- 추상화를 사용할 때의 장점과 단점을 모두 이해하고, 프로젝트 내에서 그 균형을 찾아야 함

---

# 내 생각

- 결국 추상화는 꼭 필요하지만, 모든 것이 그렇듯 극단으로 가면 좋지 않으니까, 팀 프로젝트를 할 때는 추상화를 적극 활용하자
- 추상화를 적절하게 하는 것은 굉장히 어려운 영역이고, 이는 경험을 많이 해보는 게 좋다니까 코드를 많이 보고 생각을 많이 해야겠다

---