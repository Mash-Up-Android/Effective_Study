# 32. 추상화 규약을 지켜라

---

---

# 개요

규약은 개발자들의 단순한 합의임

→ 따라서 한쪽에서 규약을 위반할 수도 있고, 기술적으로 모든 부분에서 규약 위반이 발생할 수 있음

다음과 같이 리플렉션을 활용하면 우리가 원하는 것을 열고 사용할 수 있음

```kotlin
class Employee {
    private val id: Int = 2
    override fun toString() = "User(id=$id)"

    private fun privateFunction() {
        println("Private function called")
    }
}

fun callPrivateFunction(employee: Employee) {
    employee::class.declaredMemberFunctions
        .first{ it.name = "privateFunction" }
        .apply { isAccessible = true }
        .set(employee, newId)
}

fun changeEmployeeId(employee: Employee, newId: Int) {
    employee::class.java.getDeclaredField("id")
        .apply { isAccessible = true }
        .set(employee, newId)
}

fun main() {
    val employee = Employee()
    callPrivateFunction(employee)
    // 출력 : Private Function called

    changeEmployeeId(employee, 1)
    print(employee) // 출력: User(id=1)
}
```

but 무언가를 할 수 있다는 것이 그것을 해도 괜찮다는 의미는 아님

위 코드는 `private` 프로퍼티와 `private` 함수의 이름과 같은 세부적인 정보에 크게 의존하고 있음

이러한 이름은 언제든 변경될 수 있음

따라서 이런 코드를 프로젝트에서 사용한다면, 프로젝트 내부에 시한 폭탄을 설치한 것과 같음

---

# 상속된 규약

클래스를 상속하거나, 다른 라이브러리의 인터페이스를 구현할 때는 규약을 반드시 지켜야 함

만약 규약을 제대로 지키지 않는다면, 객체가 제대로 동작하지 않을수 있음

---

# 정리

- 프로그램을 안정적으로 유지하고 싶다면 규약을 지켜야 함
- 규약을 깰 수밖에 없다면, 이를 잘 문서화 해야함

---