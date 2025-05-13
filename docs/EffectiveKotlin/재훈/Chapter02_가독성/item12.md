# 12. 연산자 오버로드를 할 때는 의미에 맞게 사용하라

---

---

# 개요

연산자 오버로딩은 굉장히 강력한 기능이지만, ‘큰 힘에는 큰 책임이 따른다’라는 말처럼 위험할 수 있음

팩토리얼을 구하는 함수 예시

```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product(): Int =
    fold(1) { acc, i -> acc * i }
```

이 함수는 Int 확장 함수로 정의되어 있으므로, 굉장히 편리하게 사용 가능함

```kotlin
print(10 * 6.factorial()) // 7200
```

팩토리얼을 !기호를 사용해 표기하는 것을 코틀린에서 다음과 같이 연산자 오버로딩을 활용하면 만들어 낼 수 있음

```kotlin
operator fun Int.not() = factorial()

print(10 * !6) // 7200
```

그러나 이렇게 해서는 안됨

함수의 이름이 not이므로 논리 연산에 사용해야지, 팩토리얼 연산에 사용하면 안 됨

코드를 이렇게 작성하면 혼란스럽고 오해의 소지가 있음

코틀린의 모든 연산자는 구체적인 이름을 가진 함수에 대한 별칭일 뿐임

| 연산자 | 대응되는 함수 |
| --- | --- |
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |
| ++a | a.inc() |
| --a | a.dec() |
| a+b | a.plus(b) |
| a-b | a.minus(b) |
| a*b | a.times(b) |
| a/b | a.div(b) |

코틀린에서 각 연산자의 의미는 항상 같게 유지됨 ⇒ 이는 매우 중요한 설계 결정임

무분별한 연산자 오버로딩의 자유는 개발자가 해당 기능을 오용하게 만듦

예를 들어 + 연산자가 일반적인 의미로 사용되지 않으면, 연산자를 볼 때마다 연산자를 개별적으로 이해해야 하기 때문에 코드를 이해하기 어려움

---

# 분명하지 않은 경우

하지만 관례를 충족하는지 아닌지 확실하지 않을 때가 문제임

예를 들어 함수를 세 배 한다는 것(* 연산자)은 무슨 의미일까?

어떤 사람은 다음과 같이 이 함수를 세 번 반복하는 새로운 함수를 만들어 낸다고 생각할 수 있음

```kotlin
operator fun Int.times(operation: () -> Unit): ()->Unit =
    { repeat(this) { operation() } }

val tripledHello = 3 * { print("Hello") }
tripledHello() // 출력: HelloHelloHello
```

어떤 사람은 다음과 같이 이러한 코드가 함수를 세번 호출한다는 것을 쉽게 이해할 수 있을 것임

```kotlin
operator fun Int.times(operation: () -> Unit) {
    repeat(this) { operation() }
}

3 * { print("Hello") }  // 출력: HelloHelloHello
```

위 코드는 함수를 생성하고, 아래 코드는 함수를 호출한다는 것이 다름

위 코드의 경우 곱셈의 결과는 ()→Unit이고, 아래 코드는 곱셈의 결과가 Unit임

의미가 명확하지 않다면, infix를 활용한 확장 함수를 사용하는 것이 좋음

일반적인 이항 연산자 형태처럼 사용 가능

```kotlin
infix fun Int.timesRepeated(operation: () -> Unit) = {
		repeat(this) { operation() }
}

val tripledHello = 3 timesRepeated { print("Hello") }
tripledHello() // 출력: HelloHelloHello
```

톱레벨 함수(클래스 또는 다른 대상 내부에 있지 않고, 가장 외부에 있는 함수)를 사용하는 것도 좋음

```kotlin
repeat(3) { print("Hello") } // 출력: HelloHelloHello
```

---

# 규칙을 무시해도 되는 경우

도메인 특화 언어(Domain Specific Language: DSL)를 설계할 때는 지금까지 설명한 연산자 오버로딩 규칙을 무시해도 됨

---

# 정리

연산자 오버로딩은 그 이름의 의미에 맞게 사용해야 함

연산자 의미가 명확하지 않다면, 연산자 오버로딩을 사용하지 않는 것이 좋음

대신 이름이 있는 일반 함수를 사용해야 함

꼭 연산자 같은 형태로 사용하고 싶다면, infix 확장 함수 또는 톱레벨 함수를 활용

---