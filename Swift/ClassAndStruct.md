Struct 는 Swift 의 값 타입을 대표한다. 변수, 상수, 함수 모두 프로퍼티로 정의할 수 있다. 프로토콜도 구현할 수 있기 때문에 POP 에도 사용 가능하다. Struct 는 다른 변수에 반영할 때 복사된다.

Class 와의 가장 큰 차이는 Class 가 참조타입이라는 것이다. 이로 인해 발생하는 차이점을 WWDC 2015 Protocol Oriented Programming in Swift 세션에서 다음과 같이 정리하였다.

1. Class 가 여러 Reference Count 를 가졌을 때 Reference Cycle 을 가지는 문제를 해결할 수 있다. Struct 는 다른 변수에 반영하면 instance 에 대한 Reference Count 가 증가하지 않고 instance 가 복사되기 때문에, Reference Cycle 이나 Race condition 등의 오류를 방지할 수 있다. 단순히 값으로써의 의미만 갖는다면 구조체가 훨씬 안전한 것이다.

2. Class 는 Class 만을 inheritance 할 수 있다. 이는 boiler-plate 코드를 줄이는 효과를 가져오지만 Class 간의 결합을 단단히 하여 유연한 코드를 작성할 수 없고, inherit 된 Class(부모)가 요구사항이 많아질수록 Code-base 가 커지는 현상을 유발한다. 상속을 통해 얻을 수 있는 강점을 struct 와 protocol 을 통해 얻을 수 있는지 검토해 보아야 하는 것이다. class 와 protocol 을 이용하는 것도 좋은 방법이다.

3. class 가 선호되는 경우는 다음과 같다. 아래 사항들은 일반적인 상황은 아니므로 struct 가 대부분의 케이스에서 선호됨을 알 수 있다.
   - 복사, 비교하지 않는 객체(예: UIWindow)
   - 외부 환경에 의해 생명주기가 결정된 객체(예: Temp Files)
   - 읽기 전용일 경우(예: CGContext)

4. stuct 가 선호되는 경우는 다음과 같다. 
   - struct 의 목적은 소량의 데이터들을 특정 도메인에 따라 저장하기 위해 주로 사용된다.

(https://stackoverflow.com/questions/24232799/why-choose-struct-over-class)

---

Instance 메서드는 클래스가 초기화 (지정 생성자를 호출) 이후에 사용할 수 있는 메서드이다.

Class 메서드는 Type 만을 가지고도 사용할 수 있는 메서드. class 선언방식(class func test() { }), static 선언방식(static func test() { })이 있는데, class 선언방식을 이용하면 override 를 할 수 없다.
