# 2. 변수의 스코프를 최소화하라

---

---

# 개요

상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋음

- 프로퍼티보다는 지역 변수를 사용하는 것이 좋음
- 최대한 좁은 스코프를 갖게 변수를 사용
    
    ex. 반복문 내부에서만 변수가 사용된다면, 변수를 반복문 내부에 작성하는 것이 좋음
    

```kotlin
// 나쁜 예
var user: User
for (i in users.indices) {
    user = users[i]
    println("User at $i is $user")
}

// 조금 더 좋은 예
for (i in users.indices) {
    val user = users[i]
    println("User at $i is $user")
}

// 제일 좋은 예
for ((i, user) in users.withIndex()) {
    println("User at $i is $user")
}
```

- 스코프를 좁게 만드는 것의 가장 중요한 이유는 프로그램을 추적하고 관리하기 쉽기 때문
    
    ⇒ 추적이 되어야 코드를 이해하고 변경하는 것이 쉬움
    
- 변수의 스코프 범위가 너무 넓으면, 다른 개발자에 의해서 변수가 잘못 사용될 수도 있음
    
    ⇒ 코드를 이해하기 굉장히 어려움
    
- 변수는 읽기 전용 또는 읽고 쓰기 전용 여부와 상관 없이, 변수를 정의할 때 초기화되는 것이 좋음
    
    ⇒ if, when, try-catch, Elvis 표현식 등을 활용하면, 최대한 변수를 정의할 때 초기화 가능
    

---

# 캡처링

소수를 구하는 에라토스테네스의 체 알고리즘 구현은 다음과 같음

```kotlin
var numbers = (2..100).toList()
val primes = mutableListOf<Int>()
while (numbers.isNotEmpty()) {
    val prime = numbers.first()
    primes.add(prime)
    numbers = numbers.filter { it % prime != 0 }
}

print(primes)
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
```

[시퀀스](https://iosroid.tistory.com/79)를 활용하는 예제로 조금 더 확장시킴

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    while (true) {
    	val prime = numbers.first()
        yield(prime) // sequence에 prime을 넘겨준다
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}

print(primes.take(10).toList())
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

이때, 위 코드와 다르게 prime을 var로 선언하고 반복문 바깥에 생성하게 되는 경우 이상한 결과가 나옴

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}

print(primes.take(10).toList())
// [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

이렇게 결과가 나온 이유는 prime 변수를 [**캡처**](https://lovia98.github.io/blog/kotlin-lamda.html)했기 때문

반복문 내부에서 filter를 활용해서 prime으로 나눌 수 있는 숫자를 필터링하는데, 시퀀스를 활용하므로 필터링이 지연됨

따라서 최종적인 prime 값으로만 필터링되고, prime이 2로 설정되어 있을 때 필터링된 4를 제외하면, drop만 동작하므로 그냥 연속된 숫자만 나와 버림

→ 가변성을 피하고 스코프 범위를 좁게 만들어서 이런 문제를 피해야함

---

# 정리

- 여러 가지 이유로 변수의 스코프는 좁게 만들어서 활용하는 것이 좋음
- var 보다는 val을 사용하는 것이 좋음
- 람다에서 변수를 캡쳐한다는 것을 꼭 기억

---