# item 15 리시버를 명시적으로 참조하라

사실 이번장의 이름은 리시버를 명시적으로 참조하라가 아니라 여러 리시버가 겹치지 않게 설계하라가 되어야 하지 않을까 싶다.  
이런식으로 리시버가 겹치면 레이블을(@객체명) 통해서 좀 우회할수 있지만 일단 헷갈린다.  

개인적으로 스코프 함수 자체를 중첩하는것을 지양해야한다고 생각하고 예시 상황의 경우 헷갈림을 강요하는 수준이라고 생각한다.  
특히 수신객체 지정람다들은 중첩하는것은 그냥 무조건 피한다.

리시버를 붙이고 안붙이고의 문제가 아니라 리시버를 참조할 필요가 없는 상황을 만들것 같다.

## DSL 마커
DSL 에서 외부 스코프의 리시버 자체를 접근 못하게 막는 
DslMarke라는 어노테이션을 소개하는데 사실 DSL자체를 진짜 잘 안만들기 때문에 있다는것만 인식할 뿐 크게 인상적이지는 않다.  
만약 DSL을 만들일이 있다면 DSL 사용 중 오용을 막는 좋은 수단 같긴하다.

컴포즈 내부 구현시에 이 DSL 마커가 적극적으로 쓰인다고 한다.  
나중에 컴포즈 내부 까보면서 공부하다보면 자연스럽게 익숙해지지 않을까?

```kotlin
@DslMarker
@Target(
    AnnotationTarget.CLASS,
    AnnotationTarget.TYPEALIAS,
    AnnotationTarget.FUNCTION,
    AnnotationTarget.PROPERTY
)
@Retention(AnnotationRetention.BINARY)
annotation class ComposeMarker
```
이게 컴포즈에서 사용되는 경우의 수라고한다.(컴포즈 내부 인터페이스 들에 붙는 어노테이션)
