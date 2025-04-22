# 11. 가독성을 목표로 설계하라

---

---

# 개요

개발자는 코드를 작성하는 것보다 읽는 데 많은 시간을 소모함

오류가 발생한 경우 오류를 찾기 위해 코드를 작성할 때보다 오랜 시간 코드를 읽음

프로그래밍은 쓰기보다 읽기가 중요함

→ 항상 가독성을 생각하면서 코드를 작성해야 함

---

# 인식 부하 감소

사실 가독성은 사람에 따라 다르게 느껴짐

일반적으로 많은 사람의 ‘경험’과 ‘인식에 대한 과학’으로 만들어진 규칙이 존재

```kotlin
// 구현 A
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
}

// 구현 B
person?.takeIf { it.isAdult }
    ?.let(view::showPerson)
    ?: view.showError()
```

B가 더 짧다는 이유로 가독성이 더 좋은 것은 아님 ⇒ 읽고 이해하기 어려움

가독성이란 코드를 읽고 얼마나 빠르게 이해할 수 있는지를 의미함

우리의 뇌가 얼마나 많은 관용구(구조, 함수, 패턴)에 익숙해져 있는지에 따라 다름

코틀린 초보자에게는 일반적인 관용구(`if`/`else`, `&&`, 메서드 호출)를 사용하는 A가 더 읽고 이해하기 쉬움

B는 코틀린에서는 꽤 일반적으로 사용되는 관용구(`?.`, `takeIf`, `let`, Elvis, 제한된 함수 레퍼런스 `view::showPerson`)을 사용하고 있음

하지만 숙련된 개발자만을 위한 코드는 좋은 코드가 아님

A와 B 중 A가 훨씬 가독성이 좋은 코드임

코드 수정해야 하는 경우

- A는 if 블록에 작업을 추가하면 되므로 쉽게 수정 가능하고, 디버깅도 더 쉬움
- B는 더 이상 함수 참조를 사용할 수 없어  코드를 수정해야 하고, else 블록 수정이 어려움
    
    Elvis 연산자의 오른쪽 부분이 하나 이상의 표현식을 갖게 하려면, 함수를 추가로 사용해야 함
    

```kotlin
// 구현 A 수정
if(person != null && persion.isAdult) {
    view.showPerson(person)
    view.hideProgressWithSuccess()
} else {
    view.showError()
    view.hideProgress()
}

// 구현 B 수정
person?.takeIf { it.isAdult }
    ?.let {
        view.showPerson(it)
        view.hideProgressWithSuccess()
    } ?: run {
        view.showError()
        view.hideProgress()
    }
```

A와 B는 실행 결과가 다름

let은 람다식의 결과를 리턴함

즉, showPerson이 null을 리턴하면, 두 번째 구현 때는 showError도 호출함

‘인지 부하’를 줄이는 방향으로 코드를 작성하라

우리의 뇌는 패턴을 인식하고, 패턴을 기반으로 프로그램 작동 방식을 이해함

가독성은 ‘뇌가 프로그램의 작동 방식을 이해하는 과정’을 더 짧게 만드는 것

자주 사용되는 패턴을 활용하면, 이 과정을 더 짧게 만들 수 있음

결론 → 짧은 코드 보다 익숙한 코드가 최고

---

# 극단적이 되지 않기

위에서 let으로 인해 예상치 못한 결과가 나온다 했지만, ‘let은 절대로 쓰면 안 된다’처럼 극단적이 되지 말아야 함

let은 좋은 코드를 만들기 위해 다양하게 활용되는 관용구임

예를 들어 nullable 가변 프로퍼티가 있고, null이 아닐 때만 어떤 작업을 수행해야 하는 경우

가변 프로퍼티는 쓰레드와 관련된 문제를 발생시킬 수 있으므로, 스마트 캐스팅이 불가능함

여러 해결 방법이 있는데, 일반적으로 다음과 같이 안전 호출 let을 사용

```kotlin
class Person(val name: String)
val person: Person? = null

fun printName() {
    person?.let {
        print(it.name)
    }
}
```

이외에도 다음과 같은 경우에 let을 많이 사용

- 연산을 아규먼트 처리 후로 이동시킬 때
- 데코레이터를 사용해서 객체를 랩할 때

```kotlin
// print 연산을 아규먼트 처리 후로 이동시키기 전
print(students.filter{}.joinToString{})

// print 연산을 아규먼트 처리 후로 이동시킨 후
students
    .filter { it.result >= 50 }
    .joinToString(separator = "\n") {
        "${it.name} ${it.surname}, ${it.result}"
    }
    .let(::print)

// 데코레이터를 사용해서 객체를 랩할 때
var obj = FileInputStream("/file.gz")
    .let(::BufferedInputStream)
    .let(::ZipInputStream)
    .let(::ObjectInputStream)
    .readObject() as SomeObject
```

이 코드들은 디버그하기 어렵고, 이해하기 어려워서 비용이 발생함

하지만 이 비용은 지불할 만한 가치가 있으므로 사용해도 괜찮음

문제가 되는 경우는 비용을 지불할 만한 가치가 없는 코드에 비용을 지불하는 경우임

어떤 것이 비용을 지불할 만한 코드인지 아닌지는 항상 논란이 있을 수 있으므로 균형을 맞추는 것이 중요함

일단 어떤 구조들이 어떤 복잡성을 가져오는지 등을 파악하는 것이 좋음

두 구조를 조합해서 사용하면, 단순하게 개별적인 복잡성의 합보다 훨씬 커짐

---

# 컨벤션

많은 개발에서 함수 이름을 어떻게 지어야 하는지, 어떤 것이 명시적이어야 하는지, 어떤 것이 암묵적이어야 하는지, 어떤 관용구를 사용해야 하는지 등으로 토론함

→ 프로그래밍은 표현력의 예술임

저자가 생각하는 최악의 코드 예시

```kotlin
val abc = "A" { "B" } and "C"
print(abc) // ABC
```

위 코드가 가능하게 하려면, 다음과 같은 코드가 있어야 함

```kotlin
operator fun String.invoke(f: ()->String): String = this + f()

infix fun String.and(s: String) = this + s
```

→ 이 코드는 수많은 규칙들을 위반함

- 연산자는 의미에 맞게 사용해야 함
    
    ⇒ invoke를 이러한 형태로 사용하면 안됨
    
- '람다를 마지막 아규먼트로 사용한다'라는 컨벤션을 여기에 적용하면, 코드가 복잡해 짐
    
    ⇒ invoke 연산자와 함께 이러한 컨벤션을 적용하는 것은 신중해야 함
    
- 위 코드에서 and라는 함수 이름이 실제 함수 내부에서 이루어지는 처리와 맞지 않음
- 문자열을 결합하는 기능은 이미 언어에 내장되어 있는데, 이미 있는 것을 다시 만들 필요는 없음

---

# 읽고 난 후

짧은 코드가 가독성이 좋은 코드이지 않나?라는 생각이 있었는데, 익숙한 코드를 짜려고 노력하자

코드의 가독성은 정답이 없는 주관적인 영역인 것 같음 → 이런 부분을 컨벤션으로 맞춰나가는게 가장 중요하겠다

---