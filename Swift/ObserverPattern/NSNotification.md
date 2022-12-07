NotificationCenter 동작 방식과 활용 방안에 대해 설명하시오.

싱글톤 객체인 NotificationCenter 에 옵저버를 등록하고 NSNotification 을 Post하게 되면 등록된 옵저버로 Notification 을 Broadcasts 한다. keyboard 관련 숨김처리 같은 미리 정의된 Notification 들에 대해서는 따로 메소드를 구현하지 않고 사용하면 최적화된 기능을 구현할 수 있다. (iOS 시스템 내부적으로 사용하고 있기 때문)
