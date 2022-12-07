Delegate란 무언인가 설명하고, retain 되는지 안되는지 그 이유를 함께 설명하시오.

retain = 클래스가 인스턴스화 될 때 메모리 내에 발생.
Class가 Delegate 할 때 사용하는 Protocol 은 클래스에 채택될 때 Class-Only-Protocol 에 의해 Class타입이 된다. 그러므로, Delegate 는 Retain 을 일으킨다.
그렇기 때문에 delegate 객체를 가리키는 변수를 weak 로 선언하는 경우가 자주 있다.

---

