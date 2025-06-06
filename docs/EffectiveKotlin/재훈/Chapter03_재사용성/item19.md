# 19. knowledge를 반복하여 사용하지 말라

---

---

# 개요

필자가 생각하는 프로그래밍의 가장 큰 규칙

> 프로젝트에서 이미 있던 코드를 복사해서 붙여넣고 있다면, 무언가가 잘못된 것이다.
> 

이를 ‘knowledge를 반복하여 사용하지 말라’라는 규칙으로 표현함

“실용주의 프로그래머”라는 책에서 ‘Don’t Repeat Yourself’라는 규칙을 ‘DRY 규칙’이라고 표현함

WET 안티패턴 : 개발자는 타이핑하는 것을 좋아하므로(비꼬는 표현), 같은 코드를 두 번씩 작성하는 의미없는 행동을 하는 것

하지만 위 이야기가 너무 자주 하는 이야기다 보니 오용되고 남용됨

이를 이해하려면 간단한 이론을 알아야 함

---

# knowledge

프로그래밍에서 knowledge는 넓은 의미로 ‘의도적인 정보’를 의미함

이와 같은 knowledge는 코드 or 데이터로 표현 가능

또한 기본 동작을 하게 아예 코드와 데이터를 부족하게 만들어서도 표현 가능

상속을 하는데도 불구하고 특정 메서드를 오버라이드하지 않게 강제한다는 것은, ‘해당 메서드가 슈퍼클래스와 동일하게 동작하기 원한다’는 의미

(ex. 클래스를 상속할 수 있게 열어두었는데, 그 안의 어떤 메서드는 오버라이드(재정의) 하지 못하게 `final`로 선언하는 경우)

이처럼 프로젝트를 진행할 때 정의한 모든 것이 knowledge임

knowledge의 종류는 다양한데, 알고리즘의 작동 방식, UI의 형태, 우리가 원하는 결과 등이 모두 ‘의도적인 정보’이며, knowledge임

knowledge는 코드, 설정, 템플릿 등으로 표현 가능

이런 knowledge는 어떤 도구, 가상머신, 다른 프로그램들에서 직접 또는 간접적으로 이해할 수 있는 정보임

프로그램에서 중요한 knowledge는 크게 두 개임

1. 로직(logic) : 프로그램이 어떤 식으로 동작하는지와 프로그램이 어떻게 보이는지
2. 공통 알고리즘(common algorithm) : 원하는 동작을 하기 위한 알고리즘

둘의 가장 큰 차이점은 **시간에 따른 변화**임

비즈니스 로직은 시간이 지나면서 계속 변하지만, 공통 알고리즘은 한 번 정의된 이후에는 크게 변하지 않음

공통 알고리즘을 최적화 or 같은 카테고리의 다른 알고리즘으로 바꿀 순 있지만, 동작은 크게 변하지 않음

---

# 모든 것은 변화한다

프로그래밍에서 유일하게 유지되는 것은 ‘변화한다는 속성’이라는 말이 있음

기술 뿐만 아니라 언어도 빠른 속도로 변화함

변화할 때 가장 큰 적은 knowledge가 반복되어 있는 부분임

프로그램에서 여러 부분에 반복되어 있는 코드를 변경하려면 검색 중 실수가 발생할 수 있고, 귀찮으며, 일부가 과거에 약간 다른 방식으로 이미 변경되었을 수 있음

⇒ knowledge의 반복은 프로젝트의 확장성(scalable)을 막고, 쉽게 깨지게(fragile) 만듦

---

# 언제 코드를 반복해도 될까?

반대로 knowledge 반복을 줄이면 안되는 상황은, 얼핏보면 knowledge 반복처럼 보이지만, 실직적으로 다른 knowledge를 나타내므로 추출하면 안되는 부분임

신중하지 못한 추출은 변경을 더 어렵게 만들고, 구성을 읽을 때도 더 어려울 수 있음

한 가지 유용한 휴리스틱으로, 비즈니스 규칙이 다른 곳(source)에서 왔는지 확인하는 방법이 있음

다른 곳에서 왔다면, 독립적으로 변경될 가능성이 높고, 잘못된 코드 추출로부터 보호할 수 있는 규칙도 있음

이는 바로 단일 책임 원칙(Single Responsibility Principle, SRP)임

---

# 단일 책임 원칙

코드를 추출해도 되는지를 확인할 수 있는 원칙으로, SOLID 원칙 중 하나인 다일 책임 원칙이 있음

단일 책임 원칙이란 ‘클래스를 변경하는 이유는 단 한 가지여야 한다’라는 의미

ex. 두 액터가 같은 클래스를 변경하는 일은 없어야 한다

→ 액터는 변화를 만들어내는 존재, 서로 업무와 분야에 대해서 잘 모르는 개발자로 비유됨

---

# 정리

- 모든 것은 변화하므로, 공통 knowledge가 있다면 이를 추출해서 변화에 대비해야 함
- 여러 요소에 비슷한 부분이 있는 경우, 변경이 필요할 때 실수가 발생할 수 있으므로 추출하는게 좋음
- 의도하지 않은 수정을 피하거나, 다른 곳에서 조작하는 부분이 있다면, 분리해서 사용하는 게 좋음
- DRY(자기복제)를 피하기 위해 비슷해 보이는 코드를 극단적으로 추출하려하는 것은 좋지 않음

---