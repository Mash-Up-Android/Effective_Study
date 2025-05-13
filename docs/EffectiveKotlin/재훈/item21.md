# 21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

---

---

# 개요

코틀린은 코드 재사용과 관련해서 프로퍼티 위임이라는 새로운 기능을 제공함

프로퍼티 위임을 사용하면 일반적인 프로퍼티의 행위를 추출해서 재사용 가능

대표적인 예로 지연 프로퍼티가 있는데, lazy 프로퍼티는 이후에 처음 사용하는 요청이 들어올 때 초기화되는 프로퍼티를 의미함

이러한 패턴은 굉장히 많이 사용되는데, 대부분 언어에서는 필요할 때마다 구현해야 하지만, 코틀린에서는 프로퍼티 위임을 활용해 간단하게 구현 가능

코틀린의 stdlib는 lazy 프로퍼티 패턴을 쉽게 구현할 수 있게 `lazy`함수를 제공함

```kotlin
val value by lazy { createValue() }
```

프로퍼티 위임을 사용하면, 이외에도 변화가 있을 때 이를 감지하는 observable 패턴을 쉽게 만들 수 있음

목록을 출력하는 리스트 어댑터가 내부 데이터가 변경될 때마다 변경된 내용을 다시 출력하는 경우 + 프로퍼티의 변경 사항을 로그로 출력하고 싶은 경우

⇒ 다음과 같이 stdlib의 `observable` 델리게이트를 기반으로 간단하게 구현 가능

```kotlin
// 내부 데이터가 변경될 때마다 변경된 내용을 다시 출력
val item: List<Item> by
    Delegates.observable(listOf()) { _, _, _ -> 
        notifyDataSetChanged()
}

// 프로퍼티 변경사항을 로그로 출력
var key: String? by Delegates.observable(null) {  _, old, new -> 
    Log.e("key changed from $old to $new")
}
```

일반적으로 프로퍼티 위임 매커니즘을 활용하면, 다양한 패턴들을 만들 수 있음

(ex. 뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩 등)

일반적으로 이런 패턴들을 사용할 때 자바에서는 어노테이션을 많이 활용해야 하지만, 코틀린은 프로퍼티 위임을 사용해서 간단하고 type-safe하게 구현 가능

```kotlin
// 안드로이드에서의 뷰와 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val textSize by bindDimension(R.dimen.font_size)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

// Kotlin에서의 종속성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModel by viewModel()

// 데이터 바인딩
private val port by bindConfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```

프로퍼티 델리게이트로, 일부 프로퍼티가 사용될 때 간단한 로그를 출력하는 예제

```kotlin
var token: String? = null
    get() {
        println("token returned value $field")
        return field
    }
    set(value) {
        println("token changed value from $field to $value")
        field = value
    }

var attempts: Int = 0
    get() {
        println("attempts returned value $field")
        return field
    }
    set(value) {
        println("attempts changed value from $field to $value")
        field = value
    }
```

두 프로퍼티는 타입이 다르지만, 내부적으로 거의 같은 처리를 하고, 프로젝트에서 자주 반복되는 패턴임

⇒ 따라서 프로퍼티 위임을 활용해서 추출하기 좋음

프로퍼티 위임은 다른 객체의 메서드를 활용해서 프로퍼티의 접근자(게터/세터)를 만드는 방식임

다른 객체의 메서드 이름이 중요한데, 게터는 `getValue`, 세터는 `setValue` 함수를 사용해서 만들어야 함

객체를 만든 뒤에는 `by` 키워드를 사용해서, `getValue`, `setValue`를 정의한 클래스와 연결해주면 됨

위 예제를 프로퍼티 위임을 활용해 변경한 예제

```kotlin
var token: String? by LoggingProperty(null)
var attempts: Int by LoggingProperty(0)

private class LoggingProperty<T>(var value: T) {
    operator fun getValue(
        thisRef: Any?,
        prop: KProperty<*>
    ): T {
        println("${prop.name} returned value $value")
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        prop: KProperty<*>,
        newValue: T
    ) {
        val name = prop.name
        println("$name changed from $value to $newValue")
        value = newValue
    }
}
```

프로퍼티 위임이 어떻게 동작하는지 이해하려면, by가 어떻게 컴파일되는지 보는 것이 좋음

위의 코드에서 `token` 프로퍼티는 다음과 비슷한 형태로 컴파일 됨

```kotlin
@JvmField
private val `token$delegate` = LoggingProperty<String?>(null)

var token: String?
    get() = `token$delegate`.getValue(this, ::token)
    set(value) {
        `token$delegate`.setValue(this, ::token, value)
    }
```

위 코드에서 token 프로퍼티는 다음과 비슷한 형태로 컴파일 됨

단순히 `getValue`/`setValue`가 값만 처리하는 형태가 아니라 컨텍스트(`this`)와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 바뀜

프로퍼티에 대한 레퍼런스는 이름, 어노테이션과 관련된 정보 등을 얻을때 사용됨

컨텍스트는 함수가 어떤 위치에서 사용되는지와 관련된 정보를 제공해 줌

이러한 정보로 인해서 `getValue`와 `setValue` 메서드가 여러 개 있어도 문제 없음

`getValue`와 `setValue` 메서드가 여러 개 있어도 컨텍스트를 활용하므로, 상황에 따라 적절한 메서드가 선택됨

이는 굉장히 다양하게 활용 되는데, 예를 들어 여러 종류의 뷰와 함께 사용할 수 있는 델리게이트가 필요한 경우, 다음과 같이 구현해서 컨텍스트의 종류에 따라서 적절한 메서드가 선택되게 만들 수 있음

```kotlin
class SwipeRefreshBinderDelegate(Val id: Int) { 
    private var cache: SwipeRefreshLayout? = null

    operator fun getValue(
    	activity: Activity,
        prop: KProperty<*>
    ): SwipeRefreshLayout {
    return cache ?: activity
        .finViewById<SwipeRefreshLayout>(id)
        .also { cache = it }
    }

    operator fun getValue(
    	fragment: Fragment,
        prop: KProperty<*>
    ): SwipeRefreshLayout {
    	return cache ?: fragment.view
            .findViewById<SwipeRefreshLayout>(id)
            .also { cache = it }
     }
}
```

객체를 프로퍼티 위임하려면 `val`은 `getValue` 연산, `var`은 `getValue`와 `setValue` 연산이 필요함

이러한 연산은 지금까지 살펴본 것처럼 멤버 함수로도 만들 수 있지만, 확장 함수로도 만들 수 있음

`Map<String, *>`를 사용하는 예제

```kotlin
val map: Map<String, Any> = mapOf(
    "name" to "Marcin",
    "kotlinProgrammer" to true
)
val name by map
print(name) // Marcin
```

이는 코틀린 stdlib에 다음과 같은 확장 함수가 정의되어 있어서 사용할 수 있음

```kotlin
inline operator fun <V, V1 : V> Map<in String, V>
.getValue(thisRef: Any?, property: KProperty<*>): V1 =
getOrImplicitDefault(property.name) as V1
```

코틀린 stdlib에서 다음과 같은 [프로퍼티 델리게이터](https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-%EB%82%B4%EC%9E%A5%EB%90%9C-delegates-2%ED%8E%B8-bc4a23cb6f10)를 알아 두면 좋음

- `lazy` : 늦은 초기화
- `Delegates.observable` : 변경 감지
- `Delegates.vetoable` : 값의 변경에 대한 통지 거부권을 가짐
- `Delegates.notNull` : set 하려는 값이 null일 경우 예외 발생

굉장히 범용적으로 사용되는 패턴들에 대한 프로퍼티 델리게이터이므로 알아두면 좋음

또한 프로퍼티 델리게이터를 직접 만들어서 사용할 수도 있음

---

# 정리

- 프로퍼티 델리게이트는 프로퍼티와 관련된 다양한 조작이 가능하고, 컨텍스트와 관련된 대부분의 정보를 가짐
    
    이러한 특징으로 인해 다양한 프로퍼티의 동작을 추출해서 재사용 가능
    
    ⇒ 표준 라이브러리의 `lazy`와 `observable`이 대표적인 예
    
- 프로퍼티 위임은 프로퍼티 패턴을 추출하는 일반적인 방법이라 많이 사용됨
    
    따라서 코틀린 개발자라면 프로퍼티 위임이라는 강력한 도구를 잘 알고 있어야 함
    
    ⇒ 일반적인 패턴을 추출하거나 더 좋은 API를 만들 때 활용 가능
    

---