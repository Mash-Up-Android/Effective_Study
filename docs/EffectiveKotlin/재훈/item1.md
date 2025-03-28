# 1. 가변성을 제한하라

---

---

# 개요

코틀린은 읽고 쓸 수 있는 프로퍼티(read-write property) `var`를 사용하거나 mutable 객체를 사용하면 상태를 가질 수 있음

상태를 적절하게 관리하는 것이 어려움

1. 프로그램을 이해하고 디버그하기 힘들어짐
    
    ⇒ 상태를 갖는 부분들의 관계를 이해해야 하며, 상태 변경이 많아지면 이를 추적하기 힘들어짐. 이러한 클래스를 이해하기도 어렵고, 이후에 코드를 수정하기도 힘듦. 클래스가 예상하지 못한 상황 또는 오류를 발생시키는 경우에 큰 문제가 됨
    
2. 가변성(mutability)이 있으면, 코드의 실행을 추론하기가 어려워짐
    
    ⇒ 시점에 따라 값이 달라질 수 있으므로, 현재 어떤 값을 갖고 있는지 알아야 코드의 실행을 예측할 수 있음. 한 시점에 확인한 값이 계속 동일하게 유지된다고 확신할 수도 없음.
    
3. 멀티스레드 프로그램일 때는 적절한 동기화가 필요함
    
    ⇒ 변경이 일어나는 모든 부분에서 충돌이 발생 가능
    
4. 테스트하기 어려움
    
    ⇒ 모든 상태를 테스트해야 하므로, 변경이 많으면 더 많은 조합을 테스트 해야함
    
5. 상태 변경이 일어날 때, 다른 부분에 이런 변경을 알려야하는 경우가 있음
    
    ⇒ 예를들어, 정렬된 리스트에 가변 요소를 추가한다면, 리스트 전체를 다시 정렬해야 함
    

---

# 코틀린에서 가변성 제한하기

코틀린에서 가변성을 제한하기 위해 가장 많이 사용되고 중요한 방법 3가지

- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분하기
- 데이터 클래스의 copy

## 읽기 전용 프로퍼티(val)

val을 사용해 읽기 전용 프로퍼티를 만들 수 있음

이렇게 선언된 프로퍼티는 값(value)처럼 동작하며, 일반적인 방법으로는 값이 변하지 않음

읽기 전용 프로퍼티는 완전 변경 불가능한 것이 아닌, mutable 객체를 담고 있다면 내부적으로 변할 수 있음

```kotlin
val list = mutableListOf(1,2,3)
lsit.add(4)

print(list) // [1, 2, 3, 4]
```

읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의 가능

이렇게 var 프로퍼티를 사용하는 val 프로퍼티는 var 프로퍼티가 변할 때 변할 수 있음

```kotlin
var name: String = "Marchin"
var surname: String = "Moskala"
val fulName
		get() = "$name $surname"

fun main() {
		println(fullName) // Marcin Moskala
    name = "Maja"
    println(fullName) // Maja Moskala
}
```

코틀린의 프로퍼티는 기본적으로 캡슐화 되어 있고 추가적으로 사용자 정의 접근자(게터, 세터)를 가질 수 있음

이러한 특성으로 코틀린은 API를 변경하거나 정의할 때 굉장히 유연함

var은 **게터**와 **세터**를 모두 제공하지만, val은 변경 불가능하므로 **게터**만 제공함

그래서 val은 var로 오버라이드 가능

```kotlin
interface Element {
		val active: Boolean
}

class ActualElement: Element {
		override var active: Boolean = false
}
```

val 값은 변경될 수 있기는 하지만, 프로퍼티 레퍼런스 자체를 변경할 수는 없으므로 동기화 문제 등을 줄일 수 있음 

→ 그래서 일반적으로 var보다는 **val**을 많이 사용함

val은 읽기 전용 프로퍼티지만, 변경할 수 없음을 의미하는 것은 아님

## 가변 컬렉션과 읽기 전용 컬렉션 구분하기

프로퍼티와 마찬가지로, 코틀린은 **읽고 쓸 수 있는 컬렉션**과 **읽기 전용 컬렉션**으로 구분

- 읽고 쓸 수 있는 컬렉션 : MutableIterable, MutableCollection, MutablSet, MutablList
- 읽기 전용 컬렉션 : Iterable, Collection, Set, List

mutable이 붙은 인터페이스는 대응되는 읽기 전용 인터페이스를 상속 받아서, 변경을 위한 메서드를 추가한 것

→ 읽기 전용 프로퍼티가 게터만 갖고, 읽고 쓰기 전용 프로퍼티는 게터와 세터를 모두 갖는 것과 비슷하게 동작

읽기전용 컬렉션을 mutable 컬렉션으로 변경해야 한다면, copy를 통해서 새로운 mutable 컬랙션을 만드는 `list.toMutableList` 를 활용해야 함

## 데이터 클래스의 copy

immutable 객체의 장점

1. 한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉬움
2. immutable 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬 처리를 안전하게 가능
3. immutable 객체에 대한 참조는 변경되지 않으므로, 쉽게 캐싱 가능
4. immutable 객체는 [방어적 복사](https://aahc912.tistory.com/93)본(defensive copy)을 만들 필요 없고, 깊은 복사를 따로 하지 않아도 됨
5. immutable 객체는 다른 객체(mutable or immutable)를 만들 때 활용하기 좋음
6. immutable 객체는 Set 또는 Map의 키로 사용 가능

mutable 객체의 단점 : 예측하기 어려우며 위험함

immutable 객체의 단점 : 변경할 수 없음

따라서 immutable 객체는 자신의 일부를 수정한 새로운 객체를 만들어 내는 메서드를 가져야 함

⇒ data 클래스의 data 한정자가 `copy`라는 메서드를 만들어줌

```kotlin
data class User(
		val name: String,
		val surname: String
)

var user = User("Maja", "Markiewicz")
user = user.copy(surname = "Moskala")
print(user) // User(name=Maja, surname=Moskala)
```

---

# 다른 종류의 변경 가능 지점

변경할 수 있는 리스트를 만들어야 할 때, mutable 컬렉션을 만든느 것과 var로 읽고 쓸 수 있는 프로퍼티를 만드는 것 두 가지 선택지가 있음

```kotlin
val list1: MutablList<Int> = mutableListOf()
var list2: List<Int> = listOf()

// 1
list1.add(1)
list2 = list2 + 1

// 2
list1 += 1
list2 += 2
```

1, 2번 코드 모두 정상적으로 동작하지만, 장단점이 존재

1번

- 구체적인 리스트 구현 내부에 변경 가능 지점이 있음
- 멀티스레드 처리가 이루어질 경우, 내부적으로 적절한 동기화 되어 있는지 확실하게 알 수 없으므로 위험

2번

- 프로퍼티 자체가 변경 가능 지점임
- 따라서 멀티스레드 처리의 안정성이 더 좋음

mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 세터(`Delegates.observable`)을 사용해서 변경을 추적할 수 있음

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new ->
		println("Names changed from $old to $new")
}

names += "Fabio"
// names가 []에서 [Fabio]로 변합니다.
names += "Bill"
// names가 [Fabio]에서 [Fabio, Bill]로 변합니다.
```

mutable 컬렉션을 사용하는 것이 처음에는 더 간단하게 느껴지겠지만, mutable 프로퍼티를 사용하면 객체 변경을 제어하기가 더 쉬움

참고로 최악의 방식은 프로퍼티와 컬렉션을 모두 변경 가능한 지점으로 만드는 것

---

# 변경 가능 지점 노출하지 말기

상태를 나타내는 mutable 객체를 외부에 노출하는 것은 위험

```kotlin
data class User(val name: String)

class UserRepository {
	private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
	    	return storedUsers
    }
    
    // ...
}

val userRepository = UserRepository()

val storeUsers = userRepository.loadAll()
storeUsers[4] = "Kirill"
// ...

print(userRepository.loadAll()) // {4=Kirill}
```

loadAll을 사용해서 private 상태인 UserRepository를 수정할 수 있는데, 이러한 코드는 돌발적인 수정이 일어날 때 위험

이를 처리하는 두가지 방법은 다음과 같음

1. 리턴되는 mutable 객체를 복제하는 방어적 복제(defensive copying)를 해야 함(copy 메서드 활용)

```kotlin
class UserRepository {
		private val user: MutableUser()

    fun get(): MutableUser {
	    	return user.copy()
    }
    
    // ...
}
```

2. 가능하다면 무조건 가변성을 제한하는 게 좋음

컬렉션은 객체를 읽기 전용 슈퍼타입으로 업캐스트하여 가변성을 제한 가능

```kotlin
data class User(val name: String)

class UserRepository {
	private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): Map<Int, String> {
	    	return storedUsers
    }
    
    // ...
}
```

---

# 정리

- var보다는 val을 사용하는 것이 좋음
- mutable 프로퍼티 보다는 immutable 프로퍼티를 사용하는 것이 좋음
- mutable 객체와 클래스보다는 immutable 객체와 클래스를 사용하는 것이 좋음
- 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy를 활용하는 것이 좋음
- 컬렉션에 상태를 저장해야 한다면, mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋음
- 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋음
- mutable 객체를 외부에 노출하지 않는 것이 좋음

몇가지 예외로, 가끔 효율성 때문에 immutable 객체보다 mutable 객체를 사용하는 것이 좋을 때도 있음

immutable 객체를 사용할 때는 언제나 멀티스레드 때에 더 많은 주의를 기울여야 함

immutable 객체와 mutable 객체를 구분하는 기준은 가변성임

---