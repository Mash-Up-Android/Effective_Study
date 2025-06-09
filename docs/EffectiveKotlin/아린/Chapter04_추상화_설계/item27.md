# 변화로부터 코드를 보호하려면 추상화를 사용하라

- 추상화란, 코드의 세부 구현을 감추고, 외부에는 필요한 부분만 노출하는 것을 의미

함수, 클래스를 추상화하고 실질적인 코드를 숨기면
사용자는 세부 사항을 몰라도 괜찮고, 원하는 대로 수정할 수도 있다

서론에서 언급했던 자동차 인터페이스 처럼
운전대, 페달 등을 조작할 줄만 알면 그 자동차의 내부 구조, 작동 원리에 대해 몰라도 된다

그래서 자동차 제조업체는 더 친환경적이고, 더 많은 센서를 추가한 자동차를 안전하게 만들 수 있다

이번 장에서는 추상화를 통해 코드를 보호하는 세 가지 사례를 살펴본다

## 상수

첫 번째로 상수이다.

우리가 개발할 때 비밀번호 최소 길이 등을 상수로 빼는 것이 이와 관련되어 있다

```kotlin
const val MIN_PASSWORD_LENGTH = 7

fun isPasswordValid(text: String) : Boolean {
		if(text.length < MIN_PASSWORD_LENGTH) return false
		if(text.length < 7) return false
}
```

직접 “7”을 입력하는 것보다 “MIN_PASSWORD_LENGTH” 라는 상수로 뺌으로써
“7”의 의미를 빠르고 쉽게 이해할 수 있게 된다

비밀번호 최소 길이를 변경하기도 쉬워진다
함수 내부 로직은 몰라도, 상수 값만 변경하면 된다

따라서, 이렇게 두 번 이상 사용되는 값은 상수로 추출하자

> 상수로 추출하면,
- 의미있는 이름을 붙여 이해가 쉽다
- 나중에 해당 값을 쉽게 변경할 수 있다
>

## 함수

```kotlin
fun Context.toast(
		message: String,
		duration: Int = Toast.LENGTH_LONG
) {
		Toast.makeText(this, message, Toast.LENGTH_LONG).show()
}

// 일반적인 사용
context.toast(message)

// 액티비티 또는 Context 서브 클래스에서 사용
toast(message)
```

토스트 메시지를 띄우는 함수를 위와 같은 toast 확장 함수로 만들어서 사용할 수 있다

`Toast.makeText(this, message, Toast.LENGTH_LONG).show()`
위의 공통 코드는 확장 함수를 만들어서 일반적인 알고리즘으로 추출하자.

이렇게 하면 토스트를 출력하는 자세한 코드를 기억하지 않아도,
toast 라는 확장 함수를 통해 간단하게 구현할 수 있다

토스트를 출력하는 방법이 변경된다고 해도 확장 함수 내부 코드만 수정하면 되므로 유지보수성도 향상된다

하지만, 토스트가 아니라 스낵바를 출력하는 것으로 변경된다면 어떻게 될까?

Context.toast를 Context.snackbar로 변경하면 될까?

함수의 이름을 직접 바꾸는 것은 좋은 방법이 아니다. (아이템 28)

함수의 파라미터까지 한 번에 변경하는 것 또한 쉽지 않은 일이라, Toast.LENGTH_LONG이 계속 사용된다

그렇다면 우리는 메시지의 출력 **방법**이 아니라, **메시지를 출력하고 싶다는 의도** 자체를 나타내야 한다

```kotlin
fun Context.showMessage(
		message: String,
		duration: MessageLength = MessageLength.LONG
) {
		val toastDuration = when(duration) {
				SHORT -> Length.LENGTH_SHORT
				LONG -> Length.LENGTH_LONG
		}
		Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength {SHORT, LONG}
```

이렇게 showMEssage라는 높은 레벨의 함수로 바꿀 수 있다.

여기서 가장 큰 변화는 이름이다
큰 차이가 없다고 생각할 수 있지만, 추상화를 표현하는 함수는 의미있는 함수 이름을 나타내는 것이 가장 중요하다

함수는 단순한 추상화지만 제한도 많고, 상태를 유지하지 않는다

또한 함수 이름을 변경하면 프로그램 전체에 큰 영향을 준다

구현을 추상화하는 더 강력한 방법은 클래스이다

## 클래스

```kotlin
Class MessageDisplay(val context : Context) {
		fun show(
				message: String,
				duration: MessageLength = MEssageLenth.LONG
		) {
				val toastDuration = when(duration) {
						SHORT -> Length.LENGTH_SHORT
						LONG -> Length.LENGTH_LONG
				}
				Toast.makeText(this, message, toastDuration).show()
		}
}

enum class MessageLength {SHORT, LONG}

// 사용
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```

클래스가 더 강력한 이유는 상태를 가질 수 있다는 점, 많은 함수를 가질 수 있다는 점이다

위 코드에서 클래스 상태인 context는 기본 생성자를 통해 주입되는데,
의존성 주입 프레임워크로 클래스 생성을 위임할 수도 있다

```kotlin
@Inject lateinit var messageDisplay : MessageDisplay
```

mock 객체를 활용해 이 클래스에 의존하는 다른 클래스를 테스트 할 수도 있다

```kotlin
val messageDisplay: MessageDisplay = mockk()
```

메시지를 출력하는 더 다양한 종류의 메서드를 만들 수 있어 확장성에서도 유리하다

```kotlin
messageDisplay.setChristmasMode(true)
```

## 인터페이스

- `listOf` 함수는 `List`를 리턴하는데, 이 `List`는 인터페이스
- 컬렉션 처리 함수는 `Iterable`, `Collection`의 확장 함수로 `List`, `Map`과 같은 인터페이스를 리턴
- 함수 `lazy`는 `Lazy` 인터페이스를 리턴

라이브러리를 만드는 사람은 내부 클래스의 가시성을 제한하고 인터페이스를 노출한다

사용자가 클래스를 직접 사용하지 못하므로,
인터페이스 뒤에 객체를 숨겨 걱정없이 인터페이스의 구현을 변경하거나 유지할 수 있기 때문이다

그렇게 하여 사용자는 추상화된 것에만 의존하게 되고, 결합도가 낮아진다

또한 선언과 사용을 분리함으로써 실제 구현을 자유롭게 변경할 수 있게 된다

---

지금까지 추상화 하는 방법을 정리하면 다음과 같다

- 상수로 추출한다
- 동작은 함수로 래핑한다
- 함수를 클래스로 래핑한다
- 인터페이스 뒤에 클래스(구현)을 숨긴다
- 보편적인 객체를 특수한 객체로 래핑한다

이를 구현 할 때에는 다음과 같은 도구를 활용할 수 있다

- 제네릭 타입 파라미터
- 내부 클래스 추출
- 생성을 제한한다

### 추상화에 문제는 없을까?

추상화는 자유로운 대신 코드를 이해하고 수정하기엔 어렵다
이 코드를 읽는 사람이 코드 개념을 배우고, 잘 이해해야 한다

큰 프로젝트에서는 모듈화를 잘 해야하는 등의 비용이 발생하기 때문에 극단적인 추상화는 지양하자

너무 많은 것을 숨기게 되면 결과를 이해하는 것 자체가 어렵다

누군가는 showMessage를 토스트 출력 함수로 생각한다든지,
Toast.makeText를 따로 찾는다든지 할 수 있기 때문에 코드를 이해하기 어렵게 만들 수 있다

### 균형을 맞추자

- 팀의 크기
- 팀의 경험
- 프로젝트의 크기
- feature set
- 도메인 지식

이렇게 프로젝트에 따라 균형을 적절히 찾아가보자.