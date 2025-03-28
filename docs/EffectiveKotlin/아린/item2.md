# 변수의 스코프를 최소화하라

결론부터 말하면

>프로퍼티보다는 지역 변수를 사용하자
>
>변수가 최대한 좁은 스코프를 갖도록 하자


| **프로퍼티 (Property)** | **지역 변수 (Local Variable)** |
| --- | --- |
| 클래스 또는 객체 내부에서 선언 | 함수 내부에서 선언 |
| 클래스의 인스턴스가 존재하는 동안 유지됨 | 선언된 함수 또는 블록 내에서만 사용 가능 |
| 객체가 살아있는 동안 유지됨 | 함수 실행이 끝나면 사라짐 |
| 클래스의 상태를 저장하고 공유 | 함수 내부에서 일시적으로 값 저장 및 연산 수행 |
| 클래스 인스턴스를 통해 접근  | 함수 내부에서만 직접 접근 가능 |

다음 코드는 나쁜 예시다

```kotlin
var user: User
for (i in users.indices) {
    user = users[i]
    print("User at $i is $user")
}
```

**변수 user가 루프 밖에서 선언되었다**

루프가 반복될 때마다 이전 반복에서 사용한 값이 남아 있게 된다.

불필요한 변수 할당을 하게 되고, 루프가 끝난 후에도 user가 남아있어 불필요한 메모리를 차지할 수 있다

루프 내부에서만 사용되는 변수를 루프 외부에 선언하면 코드의 가독성이 떨어지고, 예상치 못한 버그가 발생할 가능성이 높아진다

### **좋은 예 (스코프를 줄임)**

```kotlin
for (i in users.indices) {
    val user = users[i]
    print("User at $i is $user")
}
```

**변수 user를 루프 내부에서만 사용하도록 변경되었다**

user가 val로 루프 내부에서 선언되어 각 반복마다 새로 생성되어, 이전 값과 관계가 없어진다

user는 루프 내부에서만 접근할 수 있으므로, 불필요한 변수 할당을 줄이고 가독성이 향상된다

루프가 끝난 후에는 user가 더 이상 존재하지 않아 메모리 관리에 유리하다

## 최적화

```kotlin
for ((i, user) in users.withIndex()) {
    print("User at $i is $user")
}
```

**withIndex()를 사용하여 i와 user를 동시에 추출한다**

users[i] 같은 인덱스 조회가 필요 없어 코드가 더 깔끔해졌고 리스트 탐색을 효율적으로 한다

가독성이 좋다! 유지보수 굿

> 스코프를 좁게 만들면 프로그램을 추적하고 관리하기 쉽다
>
> ex ) mutable 프로퍼티 변경을 추적하기 쉽다
>
> 다른 개발자들이 코드를 이해하기 쉽다

## **변수는 정의할 때 초기화하라**

초기화 시 `if`, `when`, `try-catch` 등을 활용할 수 있다

여러 프로퍼티를 한 번에 설정해야 하는 경우, **구조분해 선언**을 활용하자

```kotlin
val (description, color) = when {
    degrees < 5 -> "cold" to Color.BLUE
    degrees < 23 -> "mild" to Color.YELLOW
    else -> "hot" to Color.RED
}
```

`description`과 `color` 두 개의 변수를 한 줄에 동시에 선언하고 초기화, 각각 description, color로 분해해서 할당하는 방법

`to` 연산자: `Pair` 객체를 생성하는 **코틀린의 단축 문법**

- Pair<String, Color> 타입의 객체로 만들었음.

== Pair("cold", Color.BLUE)

## **구조 분해 선언을 해야하는 이유**

1. **코드 가독성 향상**

   Pair나 Triple 같은 복합 데이터를 다룰 때, .first, .second 같은 표현을 쓰지 않아도 됨

   변수 이름을 직관적으로 지정하여 의미를 쉽게 파악 가능

2. **데이터를 한 번에 추출**

   `Pair` 또는 `Triple` 형태로 리턴되는 데이터를 별도의 변수에 나눠서 저장할 필요 없이 한 줄로 처리


# 변수의 스코프 범위가 넓으면 캡처링 문제가 발생한다

```kotlin
package 이펙티브코틀린

fun main() {
    var numbers = (2..100).toList()
    val primes = mutableListOf<Int>()
    while (numbers.isNotEmpty()) {
        val prime = numbers.first()
        primes.add(prime)
        numbers = numbers.filter { it % prime != 0 }
    }
    println(primes)
}
```

`filter` 사용으로 매 반복마다 리스트를 새로 생성한다 → 리스트가 계속 재할당됨

더 최적화하려면 MutableList를 활용..?

`numbers.removeIf { it % prime == 0 }`

이렇게? 흠

### **캡처링 문제란?** `람다 내부`에서 `외부 변수를 참조`할 때 발생하는 문제

외부 변수가 변경될 경우, 예기치 않은 동작이 발생할 수 있다

특히 멀티스레드 환경에서는 동기화 문제가 발생할 수도..

다음 코드를 보자

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }
    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 } // 여기서 기존의 numbers가 변함!
    }
}

println(primes.take(10).toList())
```

numbers는 새로운 필터링된 시퀀스로 재할당 되고 이 numbers가 sequence {} 내부의 람다에서 캡처되는 문제.

`val` numbers로 수정하기

요 부분은 아직 잘 와닿지 않는다.. ㅜ