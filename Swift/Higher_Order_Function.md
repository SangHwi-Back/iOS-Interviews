고차함수를 뜻함.

eager와 lazy = eager-order란 각 단계의 요소마다의 연산을 통해 새로운 컬렉션을 만드는 것을 말하고, lazy-order란 요소 하나하나에 지정한 연산(연산은 여러개일 수 있음)을 한꺼번에 적용해서 최종 연산이 이뤄지기 전엔 실제 연산을 하지 않는 것을 말함.

예시
```swift 
(1...1000)
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .prefix(2)

// 1000번의 filter -> 500번의 map 연산
```

```swift
(1...1000).lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .prefix(2)

// 1일 때 filter -> 2일 때 filter, map, prefix -> 3일 때 filter -> 4일 때 filter, map, prefix. 종료.
```

LazySequenceProtocol = 해당 프로토콜을 Conform하는 컬렉션은 모두 lazy 프로퍼티를 갖는다. lazy-order 동작을 구현하고 싶을 땐 이 프로퍼티를 사용하면 된다. eager-order 동작을 사용한다면 lazy 프로퍼티 없이 사용하면 된다.

Swift에서 매개변수로 함수를 갖는 함수를 Higher-Order-Function, 고차함수라고 한다. 가장 대표적인 예시로는 map, filter, reduce 등이다. 참고로 딕셔너리는 자신의 값 만을 위한 mapValue, compactMapValue, flatMapValue 를 따로 가지기도 한다. 이 함수를 사용할 수 있는 객체는 Array, Dictionary, Set 등인데 Sequence나 Collection을 구현하는 모든 객체는 가능하다. 참고로 Collection은 이미 Sequence 프로토콜을 구현하고 있기 때문에 둘 모두를 구현할 필요는 없다.

map = 컨테이너의 값을 변경하지 않고 각 요소를 전달받은 함수를 이용해 변경한 뒤 새로운 컨테이너로 변경시킨다. 그러므로, map은 기존 데이터를 transform하는 데 많이 사용된다. (전달하는 매개변수인 함수를 map 내부에서 transform으로 네이밍 하기도 하였다)

앞에서 언급한 "기존 컨테이너는 바뀌지 않고 새로운 컨테이너로 변경시킨다."는 언급은 상당히 중요한데, 다중 스레드 환경일 때 하나의 컨테이너를 변경(물론 이렇게 코딩하면 안된다)하면 Race Condition을 유발한다. 즉, Race Condition방지 등의 안전장치가 있는 셈이다. 코드 재사용 측면에서도 생각해볼 법 한데, 같은 작업을 수행하는 map 함수를 여러번 호출해야 할 경우 배열의 Element 타입을 매개변수로 받아서 Element 타입으로 반환하는 클로저를 준비해두기만 하면 클로저를 여러번 재사용 가능하다.
([https://github.com/apple/swift/blob/main/stdlib/public/core/Map.swift](https://github.com/apple/swift/blob/main/stdlib/public/core/Map.swift))

filter = 컨테이너 내부의 값을 걸러서 추출하는 고차함수이다. map과 마찬가지로 새로운 컨테이너에 값을 담아 반환한다. filter 함수는 배열의 Element 타입을 받아 Bool 타입을 반환하는 함수를 매개변수를 받는다. 새로운 컨테이너에 포함될 항목일 경우는 true, 아니면 false이다.
([https://github.com/apple/swift/blob/main/stdlib/public/core/Filter.swift](https://github.com/apple/swift/blob/main/stdlib/public/core/Filter.swift))

reduce = 컨테이너 내부의 콘텐츠를 하나로 합치기 위한 고차함수이다. 제네릭으로 설정한 결과 타입은 초기값의 타입이 된다. 즉, 컨테이너의 Element가 무엇이든 간에 결과 타입은 자유로울 수 있다는 것이다.
([https://github.com/apple/swift/blob/main/stdlib/public/core/Sequence.swift](https://github.com/apple/swift/blob/main/stdlib/public/core/Sequence.swift))

Reference:
스위프트 프로그래밍 3판
