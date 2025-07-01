# 생성자 대신 팩토리 함수를 사용하라

클래스의 인스턴스를 만드는 가장 일반적인 방법은 기본 생성자(primary constructor)

```kotlin
class MyLinkedList<T>(
		val head: T,
		val tail: MyLinkedList<T>?
)

val list = MyLinkedList(1, MyLinkedList(2,null))
```

이렇게 하면 리스트를 중첩해서 써야하므로, 리스트 구조를 한 눈에 파악하기 어렵다

유일한 방법은 아니고, 굉장히 다양한 생성 패턴들이 있다

생성자 역할을 대신 해 주는 함수를 팩토리 함수라고 한다

팩토리 함수를 사용해 보자

```kotlin
fun <T> myLinkedListOf(
		vararg elements: T
): MyLinkedList<T>? {
		if(elements.isEmpty()) return null
		val head = elements.first()
		val elementsTail = elements
				.copyOfRange(1, elements.size)
		val tail = myLinkedListOf(*elementsTail)
		return MyLinkedList(head, tail)
}

val list = myLinkedListOf(1, 2)
```

위처럼 내부에서 재귀적으로 알아서 리스트 구조를 만들어 준다

## 팩토리 함수를 왜 쓸까?

### 생성자와 달리, 함수에 이름을 붙일 수 있다

ArrayList(3) → 리스트 첫 번째 요소인지, 리스트 크기인지 알 수 없음

ArrayList.withSize(3) → 리스트 크기임을 바로 이해할 수 있음

또한 동일한 파라미터 타입의 생성자와 충돌을 줄일 수 있다

### 생성자와 달리, 함수가 원하는 타입을 리턴할 수 있다

다른 객체를 생성할 때 인터페이스 뒤에 실제 객체 구현을 숨긴다

stdlib 라이브러리의 listOf 함수 → List 인터페이스를 리턴
이는 플랫폼에 따라 플랫폼 빌트인 컬렉션으로 만들어진다

### 생성자와 달리, 호출될 때마다 새 객체를 만들지 않아도 된다

함수를 통해 객체를 만들면 싱글턴 패턴으로 객체를 하나만 생성하게 강제할 수 있다

최적화를 위한 캐싱 매커니즘을 사용할 수도 있다

객체를 만들 수 없을 땐 null을 리턴하게 할 수도 있다

### 아직 존재하지 않는 객체를 리턴할 수 있다

어노테이션 처리 기반 라이브러리에서 주로 쓰며,
프로젝트 빌드 없이 앞으로 만들어질 객체나, 프록시를 통해 만들어질 객체를 사용할 수 있다

### 객체 외부 팩토리 함수를 만들면 가시성 제어가 자유롭다

팩토리 함수는 파일, 모듈에서 가시성 설정이 가능하다

톱레벨 팩토리 함수를 같은 파일, 모듈에서만 접근할 수 있다 등..

```kotlin
internal fun createSecretInstance(): MySecretType
```

### 인라인으로 만들 수 있고, 파라미터를 reified로 만들 수 있다

### 생성자로 만들기 복잡한 객체도 만들 수 있다

### 원하는 때에 생성자를 호출할 수 있다

## 팩토리 함수 [1] Companion 객체 팩토리 함수

팩토리 함수 정의할 때 가장 일반적인 방법이다.

- from
- of
- valueOf
- getInstance
- newInstance
- getType
- newType

또한, companion 객체는 인터페이스로 구현할 수도 있고 클래스 상속을 받을 수도 있다

## 팩토리 함수 [2] 확장 팩토리 함수

companion 객체가 있을 때, 함수처럼 쓸 수 있는 팩토리 함수를 만들어야 할 경우 확장 함수를 쓴다

## 팩토리 함수 [3] 톱레벨 팩토리 함수

- listOf
- setOf
- mapOf

public한 톱레벨 함수는 모든 곳에서 쓸 수도 있으므로 신중하게 사용해야 한다

따라서 이름을 꼭 신중하게 짓도록 해야 한다

## 팩토리 함수 [4] 가짜 생성자

- 인터페이스를 위한 생성자를 만들고 싶을 때
- reifield 타입 아규먼트를 갖게 하고 싶을 때

이 상황을 제외하면 가짜 생성자는 진짜 생성자처럼 동작해야 한다

즉, 생성자처럼 보여야 하고 같은 동작을 해야 한다

## 팩토리 함수 [5] 팩토리 클래스의 메서드

팩토리 클래스는 클래스의 상태, 프로퍼티를 가질 수 있다