# 외부 API를 랩(wrap)해서 사용하자

외부 API 설계자가 안전하다고 하든, 말든
우리가 그것을 신뢰할 수 없다고 하면 이 API는 불안정한 것이다

이렇게 불안정한 API를 과도하게 쓰는 건 위험한 일이고, 이러한 API를 어쩔 수 없이 사용해야 한다면
최대한 이 API를 로직과 직접 결합시키지 않는 것이 좋다

따라서, 잠재적으로 불안정한 외부 라이브러리는 랩(wrap)해서 사용한다

### wrap으로 다음과 같은 자유, 안정성을 얻는다

- 문제 발생 시 래퍼(warpper)만 변경하면 되어 API 변경에 쉽게 대응할 수 있다
- 프로젝트 스타일에 맞춰 API 형태 조정이 가능하다
- 특정 라이브러리가 문제라면 래퍼를 수정해서 다른 라이브러리로 대체해 코드를 쉽게 변경할 수 있다
- 쉽게 동작을 추가하고 수정할 수 있다

### wrap은 다음과 같은 단점도 있다

- 래퍼를 따로 정의해야 한다
- 다른 개발자들은 어떤 래퍼들이 있는지 따로 확인해야 한다
- 이 래퍼들은 특정 프로젝트 내부에서만 유효하므로 스택오버플로와 같은 곳에 질문할 수 없다

이 장단점을 모두 이해한 뒤에야, wrap할 API를 결정해야 한다