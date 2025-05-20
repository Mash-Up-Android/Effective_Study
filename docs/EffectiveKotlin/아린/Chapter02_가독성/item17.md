# **이름 있는 아규먼트를 사용하라**

코틀린 함수에 여러 개의 파라미터(특히 디폴트 값이 있는 파라미터)가 있을 때,

이름 있는 아규먼트(named arguments)를 사용하면 코드의 의미가 더 명확해진다

- 함수 호출 시 파라미터의 순서에 의존하지 않고, 각 값이 어떤 역할을 하는지 명확하게 알 수 있다.
- 디폴트 파라미터가 많은 함수에서 일부 값만 바꿀 때 실수를 줄이고 가독성을 높일 수 있다

```kotlin
val separator = "|"
val text = (1..10).joinToString(separator = separator)
```

위 코드와 같이

1. 변수명으로 의미 명확히 하기
2. 이름있는 아규먼트 쓰기

⇒ 가독성이 너무 좋아짐

## 언제 사용해야 할까?

### 1. 디폴트 아규먼트

```kotlin
fun greet(name: String, greeting: String = "안녕", punctuation: String = "!") {}

greet("아딘", "하이")  // punctuation은 못 바꿈
greet(name = "아딘", punctuation = "😊")
```

디폴트 값을 가진 파라미터는 함수 이름만으로 어떤 값이 어떤 역할을 하는지 명확하지 않은 경우가 많다

이름 있는 아규먼트를 사용하면 어떤 파라미터에 어떤 값을 넣는지 분명히 알 수 있어, 실수도 줄이고 코드의 의도도 잘 드러난다

### 2. **같은 타입의 파라미터가 많은 경우**

```kotlin
fun sendEmail(to: String, message: String) { ... }

// 이름 있는 아규먼트 사용
sendEmail(to = "abc@abc.com", message = "Hello, World!")

```

파라미터가 모두 같은 타입이면, 순서를 바꿔서 잘못 입력해도 컴파일러가 잡아내지 못해 버그가 생길 수 있다

이름 있는 아규먼트를 사용하면 각 값이 어떤 역할인지 명확하게 드러나고, 실수도 예방할 수 있다

### 3. **함수 타입의 파라미터가 있는 경우**

함수 타입 파라미터(람다 등)가 여러 개일 때, 이름 없는 아규먼트를 쓰면 어떤 람다가 어떤 역할인지 코드만 보고 알기 어렵다

이름 있는 아규먼트를 쓰면 각 람다의 의미가 명확하게 드러나 가독성이 좋아진다

```kotlin
fun call(before: () -> Unit = {}, after: () -> Unit = {}) { ... }

// 이름 있는 아규먼트 사용
call(before = { println("Before") }, after = { println("After") })

```