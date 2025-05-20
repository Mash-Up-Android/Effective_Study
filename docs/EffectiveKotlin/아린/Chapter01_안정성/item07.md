# 결과 부족이 발생할 경우 Null과 Failure을 사용하라

결과 부족이 발생하는 경우

- 인터넷 연결 문제
- 조건에 맞는 첫 번째 요소가 없는 경우
- 텍스트 파싱 시 형식이 맞지 않는 경우

이러한 경우 다음과 같은 두 가지 방법으로 처리할 수 있다

### 1. null 또는 sealed 클래스 (이렇게 처리하자)

- sealed 클래스는 일반적으로 Failure라는 이름을 붙인다
- 명시적으로 처리해야 한다
- 효율적이다
- 간단하다

> null과 sealed 차이
sealed result는 추가적인 정보 전달 시 사용,
그렇지 않으면 null 처리
**Failure는 처리 시 필요한 정보를 가질 수 있다**
>

### 2. 예외 Throw (예외적인 상황에서만 사용하자)

이 방법은 정보 전달용으로 사용해서는 안되며, 예외적인 상황이 발생했을 때 사용한다

왜냐하면,

- 많은 개발자가 예외 전파 과정을 제대로 추적할 수 없기 때문
- 코틀린의 모든 예외는 unchecked 예외다
    - checked : 사용자가 반드시 처리하도록 강제되는 예외
    - unchecked : 처리하지 않아도 실행에 문제 없는 예외

  > 사용자가 예외를 처리하지 않을 수 있고, 관련 내용이 문서에도 드러나지 않을 수 있다
  실제로 API 사용시 예외 관련 사항을 단순한 메서드로 파악하기 힘들다



- 예외적인 상황 처리는 명시적인 테스트인 만큼 빠르게 동작하지 않는다
- `try-catch` 블록 내부 코드 배치시 컴파일러 최적화가 제한된다
- 예외는 놓칠 수 있고, 전체 애플리케이션을 중지시킬 수도 있다

>💡
>
>충분히 예측할 수 있는 예외일 경우 null, failuer를 사용하고
>
>예측하기 어려운 예외적 범위 오류는 throw로 처리하자
>
### null 처리는 안전 호출(safe call) 또는 Elvis 연산자 등 null 안전성 기능 활용하기

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

### Result 등 같은 공용체(union type)를 리턴한다면 when 표현식 사용

```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
		is Success -> person.age
		is Failure -> -1
}
```

> 일반적으로 예상할 수 있을 때와 없을 때로 나뉘어 두 가지 형태의 함수를 사용한다
List는 두 가지를 모두 갖고 있다
>
- get : 특정 위치의 요소 추출시 사용
    - 요소가 해당 위치에 없다면 IndexOutOfBoundsException  발생
- getOrNull : out of range 오류 발생 가능성이 있을 때 사용
    - 오류 발생시 null 리턴
- 이외 getOrDefault 선택지도 있지만 일반적으로는 getOrNull 또는 Elvis(?:) 연산자를 활용하는 것이 직관적

요소를 안전하게 추출할 거라고 생각하여 nullable을 리턴하지 말자

getOrNull, Elvis를 적절히 활용하여 리턴 값을 예측되도록 만들자

**null**과 **Failure**는 예측 가능한 오류를 안전하게 처리하는 방식이고, **throw**는 예기치 않은 오류에만 사용하는 방식