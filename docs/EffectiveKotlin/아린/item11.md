# 가독성을 목표로 설계하라

개발자는 코드 작성 시간보다 읽는 데 많은 시간을 소요한다

⇒ 코드에 오류가 발생하면 코드 작성했던 시간보다 훨씬 더 많은 시간을 들여 코드를 읽는다

**즉, 프로그래밍은 쓰기보다 읽기가 중요하므로, 개발자는**

**“가독성”이 좋은 코드를 작성해야 한다는 것이다**

다음 중 어떤 코드가 더 가독성이 좋을까?

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

나는 망설이지 않고 A를 골랐다!

코드가 한 줄 더 길긴하지만 나에게 익숙한 코드여서, 아주 빠르게 읽혔다

저자 또한, 비교조차 할 수 없을 정도로 구현 A가 가독성 좋은 코드라고 했다

코틀린 경험이 적은 신입 개발자에게 B 코드를 읽으라고 한다면 `takeIf`, `let` 등의 함수 레퍼런스를 찾는데 시간을 써야한다

이러한 코드가 3줄이라서 그렇지, 몇백 몇천줄이 되면 코드 읽기가 싫어질 것 같다

사용 빈도가 적은 관용구는 코드를 복잡하게 한다

또한, 이 관용구를 여러개 조합해서 사용하면 1+1 = 2의 복잡성이 아니라 2보다 훨씬 더 크게 증가하게 된다

A는 수정하기도 쉽다

if 블록에 작업을 추가하면 된다

반면에 B는 싹다 고쳐야하는 일이 발생할 수도 있다

else 블록을 수정하는 것 자체가 어렵기 때문에,,

아무튼 괜히 굉장히 창의적으로 코드를 짜는게 좋은 것이 아니다. 유연하지 않고, 일반적이지 않다

### 짧은 코드보다 이해하기 쉬운 코드가 더 빨리 읽힌다

```kotlin
// 구현 A
if (person != null && person.isAdult) {
		view.showPerson(person)
		view.hideProgressWithSuccess()
} else {
		view.showError()
		view.hideProgress()
}

// 구현 B
person?.takeIf { it.isAdult }
		?.let {
				view.showPerson(it)
				view.hideProgressWithSuccess()
		} ?: run {
				view.showError()
				view.hideProgress()
		}
}

```

구현 B는 람다식의 결과를 리턴하는데, showPerson이 null을 리턴하면 showError() 까지 발생하게 됨

### 그래도 극단적으로 생각하려고 하지 말자

`let`을 절대 쓰지 말라는 의미가 아니다(너무 극단적임!)

> nullable 가변 프로퍼티에 대해,
null이 아닐 때에만 어떤 작업을 수행해야 하는 경우에는
안전 호출 `let` 이 용이하다!
>

```kotlin
class Person(val name: String)
var person: Person? = null

fun printName() {
		person?.let {
				print(it.name)
		}
}
```

person이 null이 아닐 때만 호출한다.

- 연산을 아규먼트 처리 후로 이동시킬 때

```kotlin
print(students.filter{}.joinToString{}

// 이와 같이 연산을 아규먼트 처리 이후로 이동시킬 때에도 let이 용이하게 쓰일 수 있다
students.filter{}.joinToString{}.let(::print)
```

- 데코레이터를 사용해 객체를 랩핑할 때
https://refactoring.guru/ko/design-patterns/decorator

`let`을 사용하여 `person` 객체가 null이 아닐 때만 이름과 나이를 출력하는 작업

---

### 코드에 비용을 지불할 때, 그만한 가치가 있는지 생각해보자

어떤 구조에 복잡성을 가져갈 때 타당한 목적이 있으면 쓰자

그 외 목적이 없을 때에는 가독성이 좋은게 좋은 거니까,.~~!

- 연산자는 의미에 맞게 사용하자
    - invoke 사용에 주의하자
    - 연산자나 함수 이름은 우리가 직관적으로 아는 동작을 해야 해.
- 람다를 마지막 아규먼트로 사용할 경우, invoke 연산자 컨벤션에 주의하자
    - 람다 표현식은 마지막 인자일 때 밖으로 빼주기

    ```kotlin
    // 이렇게 쓰지 말고
    doSomething("hello", {
        println("람다 블럭")
    })
    
    // 이렇게
    doSomething("hello") {
        println("람다 블럭")
    }
    
    ```

  ### `invoke()`는 정말 **명확한 의미가 있을 때만!**

    - 함수처럼 쓰는 게 **정말 자연스럽고 직관적일 때만** 써야 해
    - 예: 함수 레지스트리, 계산기처럼 "호출 = 실행"이 자연스러운 객체

  `invoke()`는 **애가 함수인 척 분장하고 다니는 것**

    - 근데 **함수 분장이 너무 잘 돼서** 사람들이 헷갈림 ㅋㅋ
    - 그래서 파티에 한 명만 허용 → "정말 함수처럼 보여야 할 애만"
- 현재 코드에서 and라는 함수 이름이 실제 함수 내부 처리와 맞지 않는다

    ```kotlin
    fun String.and(other: String): String {
        return "$this $other"
    }
    ```

    - 실제론 문자열 결합 기능임.

  > "and"가 아니라 그냥 combine 같은거 써라
>
- 문자열 결합 기능은 이미 언어에 내장되어 있으므로 이미 있는 것을 굳이 다시 만들지 말자
    - joinToString 쓰자.
