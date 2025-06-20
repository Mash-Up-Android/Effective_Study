# 28. API 안정성을 확인하라

---

---

# 개요

프로그래밍에서는 안정적이고 최대한 표준적인 API를 선호하는데, 이유는 다음과 같음

1. API가 변경되고, 업데이트 했다면 여러 코드를 수동으로 업데이트 해야 됨
    
    하지만, 많은 요소가 이 API에 의존하고 있다면 변경에 대응하기 어려움
    
    이를 피하기 위해 라이브러리 버전을 유지하면 버그와 취약성에 위험
    
2. 사용자가 새로운 API를 배워야 함
    
    새로 배우지 않으면 오래된 지식 때문에 보안 문제 발생 가능
    

좋은 API를 한번에 설계할 수는 없으므로, API를 안정적으로 유지하여 균형을 맞춰야 함

API가 불안정하다면 명확하게 알려줘야 함

일반적으로 시멘틱 버저닝을 사용해서 라이브러리와 모듈의 안정성을 나타냄

버전 번호를 MAJOR, MINOR, PATCH로 나누어서 구성

각각의 부분은 0이상의 정수로 구성, 0부터 시작해서 API에 다음 같은 변경 사항이 있을때 1씩 증가

- MAJOR 버전 : 호환되지 않는 수준의 API 변경
- MINOR 버전 : 이전 변경과 호환되는 기능을 추가
- PATCH 버전 : 간단한 버그 수정

안정적인 API에 새로운 요소를 추가할 때, 아직 해당 요소가 안정적이지 않다면, 먼저 다른 브랜치에 해당 요소를 두는 것이 좋음

일부 사용자가 이를 사용하도록 허용하려면, 일단 Experimental 메타 어노테이션을 사용해서 사용자들에게 아직 해당 요소가 안정적이지 않다는 것을 알려 주는 것이 좋음

→ Experimental 메타 어노테이션을 붙이면, 사용할 때 경고 또는 오류가 출력됨

요소를 오랜 시간 동안 실험적 기능으로 유지하는 것을 두려워하면 안됨

채택 속도는 느려지지만, 더 오래 동안 좋은 API를 설계하는 데 도움이 됨

안정적인 API의 일부를 변경해야 한다면, 전환하는 데 시간을 두고 Deprecated 어노테이션을 활용해서 사용자에게 미리 알려줘야함

---

# 정리

- 사용자는 API의 안정성에 대해 알아야 함
- 안정적인 API를 사용하는 것이 좋음
- 다만 안정적이라고 생각했던 API에 예상하지 못한 변경이 일어났다면, 가장 나쁜 상황임
- 모듈과 라이브러리를 만드는 사람과 이를 사용하는 사람들 사이에 커뮤니케이션이 중요함
- 커뮤니케이션은 버전 이름, 문서, 어노테이션 등을 통해 가능
- 안정적인 API에 변경을 가할 때는 사용자가 적응할 충분한 시간을 줘야 함

---