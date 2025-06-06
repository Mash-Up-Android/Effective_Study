# 20. 일반적인 알고리즘을 반복해서 구현하지 말라

---

---

# 개요

많은 개발자는 같은 알고리즘을 여러 번 반복해서 구현함

여기서 말하는 알고리즘은 특정 프로젝트에 국한된 것이 아닌, 수학적 연산, 수집 처리 같은 별도로 모듈이나 라이브러리로 분리 가능한 것을 의미

이미 구현되어 있는 것을 활용하면, 단순하게 코드가 짧아진다는 것 이외에도 다양한 장점이 있음

- 코드 작성 속도가 빨라짐 → 호출을 한 번 하는 것이 알고리즘을 만드는 것보다 빠름
- 구현을 따로 읽지 않아도, 함수의 이름 등만 보고도 무엇을 하는지 확실하게 알 수 있음
- 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있음
- 제작자들이 한 번만 최적화하면, 이러한 함수를 활용하는 모든 곳이 최적화의 혜택을 받을 수 있음

---

# 표준 라이브러리 살펴보기

일반적인 알고리즘은 대부분 이미 다른 사람들이 정의해놨음

대표적으로 stdlib는 확장 함수를 활용해서 만들어진 굉장히 거대한 유틸리티 라이브러리임

stdlib의 함수들을 하나하나 살펴보는 것은 굉장히 어려울 수 있지만, 그럴만한 가치가 있음

자세히 살펴보지 않으면, 계속해서 같은 함수를 여러 번 만들게 됨

---

# 나만의 유틸리티 구현하기

상황에 따라서 표준 라이브러리에 없는 알고리즘이 필요할 수도 있음

→ 범용 유틸리티 함수(universal utility fuction)로 정의하는 것이 좋음

ex. 컬렉션에 있는 모든 숫자의 곱을 계산하는 라이브러리

```kotlin
fun Iterable<Int>.product() = fold(1) { acc, i -> acc * i }
```

여러 번 사용되지 않는다고 하더라도 이렇게 만드는 것이 좋음

이는 잘 알려진 수학적 개념이고, `product`라는 이름이 숫자를 곱할 거라는 것이 대부분의 개발자들이 예측할 수 있기 때문임

동일한 결과를 얻는 함수를 여러 번 만드는 것은 잘못된 일임

모든 함수는 테스트되어야 하고, 기억되어야 하며, 유지보수되어야 함

따라서 함수를 만들 때는 이러한 비용이 들어갈 수 있다는 것을 전제해야 함

따라서 필요 없는 함수를 중복해서 만들지 않게, 기존에 관련된 함수가 있는지 탐색하는 과정이 필요

코틀린의 stdlib에 정의된 대부분의 함수처럼, 앞 코드의 product도 확잠 함수로 구현되어 있음

많이 사용되는 알고리즘을 추출하는 방법으로는 톱레벨 함수, 프로퍼티 위임, 클래스 등이 있음

확장 함수는 이러한 방법들과 비교해서, 다음과 같은 장점이 있음

- 함수는 상태를 유지하지 않으므로, 행위를 나타내기 좋음 ⇒ 특히 side-effect가 없는 경우에는 더 좋음
- 톱레벨 함수와 비교해서, 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한할 수 있으므로 좋음
- 수정할 객체를 아규먼트로 전달받아 사용하는 것보다는 확장 리시버로 사용하는 것이 가독성에 좋음
- 확장 함수는 객체에 정의한 함수보다 객체를 사용할 때, 자동 완성 기능 등으로 제안이 이루어지므로 쉽게 찾을 수 있음

---

# 정리

- 일반적인 알고리즘을 반복해서 만들지 말아야 함
- 대부분 stdlib에 이미 정의되어 있을 가능성이 높으므로, stdlib을 공부해 두면 좋음
- stdlib에 없는 일반적인 알고리즘이 필요하거나 특정 알고리즘을 반복해서 사용해야 하는 경우, 프로젝트 내부에 확장 함수로 정의하는게 좋음

---