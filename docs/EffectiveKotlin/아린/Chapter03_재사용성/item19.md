# Knowledge를 반복하여 사용하지 말라

knowledge

*knowledge*는 코드, 데이터, 설정, 템플릿 등으로 표현되는 "의도적인 정보"를 의미한다

즉, 의도적인 정보를 코드나 설정 등에서 반복해서 사용하지 말라

의도적인 정보 : 개발자가 직접 작성한 지식이나 규칙

1: 코드에서의 의도적인 정보

```kotlin
if (user.age >= 18) {
    allowAlcoholPurchase()
}
```

→ "18세 이상이면 술 구매 가능"이라는 도메인 규칙

2: 설정 파일에서의 의도적인 정보

```json
{
  "maxLoginAttempts": 5
}

```

→ 로그인 시도 제한을 5회로 설정

그외) 프로그램의 동작 방식(로직), UI 형태, 알고리즘, 비즈니스 규칙 등

로직 : 프로그램이 어떤 식으로 동작하는지, 프로그램이 어떻게 보이는지 → 시간이 지나면서 계속 변화

공통 알고리즘 : 원하는 동작을 하기 위한 알고리즘, 한 번 정의되면 크게 바뀌지 않는다

일단 로직과 관련되어서 살펴보자

## 모든 것은 변화한다

회사는 사용자 요구, 습관을 더 많이 알게되고

디자인 표준도 변하며, 플랫폼과 라이브러리 및 도구 등이 계속해서 변화한다

이처럼, 모든 것은 변화하기 때문에 이에 대비해야 한다

우리는 이를 위해 knowledge 반복에 대한 위험을 방지하여야 한다

작은 변경이 있음에도 모든 knowledge를 찾아 변경해야 한다는 것은 굉장히 번거로운 일이다,,

어쨌든 knowledge가 여러 곳에 반복되면, 변경 시 모든 곳을 일관되게 수정해야 하므로 실수와 버그가 발생하기 쉽고 유지보수가 어려워진다

> knowledge 반복은
- 프로젝트 확장성을 막고, 안정성이 낮아져 쉽게 깨지게 만든다
>

### 반복 가능한 경우도 있을까?

두 코드가 같은 knowledge를 나타내는지, 다른 knowledge를 나타내는지를 냉정하게 판단

- **따로 변경될 가능성이 높은 경우**

  두 코드가 비슷해 보여도, 각각의 변경 이유가 다르다면(즉, 독립적으로 진화할 가능성이 높다면) 반복(복제)해도 괜찮다

    - 두 비즈니스 규칙이 현재는 같더라도 앞으로 따로 변경될 수 있다면 합치지 않는 것이 좋음

### 반복하면 안되는 경우

- **함께 변경될 가능성이 높은 경우**

  두 코드가 본질적으로 같은 지식(knowledge)을 표현하고, 앞으로도 항상 같이 바뀔 가능성이 높다면, 반복을 피하고 하나로 만들어야 한다

- 여러 곳에서 동일한 할인 계산 로직을 쓴다면, 그 로직이 바뀔 때 모든 곳을 일일이 수정해야 함 ⇒ 함수로 묶어 한 곳에서만 관리

# 단일 책임 원칙(Single Responsibility Principle, SRP)

클래스를 변경하는 이유는 단 하나여야 한다는 원칙

즉, 한 클래스(또는 모듈, 함수 등)는 오직 하나의 책임만 가져야 하며, 하나의 변화 이유만을 가져야 한다

하나의 책임(관심사, 목적, 지식)을 가져야 유지보수가 쉽고, 코드 중복이 줄어든다

- **서로 다른 액터(Actor, 변화의 주체, 우리로 치면 개발자)가 같은 클래스를 변경하는 일이 없어야 한다**

  인증 부서와 장학금 부서가 각각 학생 클래스의 서로 다른 기능을 위해 같은 메서드나 필드를 변경한다면, 이는 단일 책임 원칙을 위배한 것임

  이런 경우 두 부서의 요구가 독립적으로 변할 수 있으므로, 각각의 책임을 분리해야 하는 것

- **비슷해 보이지만 다른 knowledge는 분리해야 한다**

  학생의 인증 통과 여부와 장학금 자격 확인은 비슷한 로직을 사용할 수 있지만, 각 부서의 정책 변화에 따라 독립적으로 바뀔 수 있음 ⇒ 같은 함수를 재사용하기보다 각각의 책임에 맞게 분리하는 습관을 들이자

- **공통 knowledge는 추출하여 변화에 대비**

  반대로, 여러 곳에서 동일한 knowledge(예시로 할인 계산 로직)를 사용한다면, 이 부분은 하나로 추출해서 관리해야 변화 시 실수를 줄일 수 있습니다

- **`StudentIsPassingValidator`** (인증 책임)
- **`StudentQualifiesForScholarshipValidator`** (장학금 책임)

### 나쁜 예

```kotlin
if (employee.role == "Manager") {
    bonus = baseSalary * 1.2
}
```

[관리자는 20% 보너스를 받는다]는 도메인 지식이 중복으로 여러 곳에 퍼져있음

### 좋은 예

```kotlin
class Employee(val role: String, val baseSalary: Double) {
    fun calculateBonus(): Double {
        return if (role == "Manager") baseSalary * 1.2 else baseSalary
    }
}
```

[보너스 계산]이라는 책임 / [관리자는 20% 보너스]라는 지식이 Employee 클래스 하나로 모임

| 상황 | 추천 |
| --- | --- |
| 진짜 핵심 로직을 가진 클래스 | 도메인 클래스로 책임 분리 |
| 그냥 보기 좋고 표현만 깔끔히 하고 싶다 | 확장 함수 |
| 테스트용/간단한 계산용 함수 | 헬퍼 함수 |