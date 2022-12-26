## 실제 디바이스가 없을 경우 개발 환경에서 할 수 있는 것과 없는 것을 설명하시오.

[TestingontheiOSSimulator.html](https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator/TestingontheiOSSimulator.html)

이전에는 위의 링크에서 명시한대로 지원되는 부분과 지원되지 않는 부분을 명확히 한 듯 합니다.

예를 들어 이전에는 "푸시 알림" 테스트가 불가하다고 하였으나, 여러 블로그에서 시뮬레이터를 통해 푸시 알림 테스트를 하는 방법이 올라와 있습니다.

아래는 현재까지 확인된 내용만을 정리한 것입니다.

* 여러 디바이스의 화면 레이아웃 테스트는 가능하다.
* 애니메이션, 제스쳐, 화면의 해상도와 색상 등은 재현 가능하나 실제 디바이스와 다를 수 있다.
* 각종 센서를 이용할 수 없다(가속도, 기압계, 주변광, GPS 등).
* 시뮬레이터는 macOS 에서 iOS 환경을 재현한 것이기 떄문에 foreground/background 환경에 따른 작업을 100% 구현하기 어렵다.
  * 예를 들어 Memory Usage 등을 실제 디바이스를 연결하여 테스트할 때와 시뮬레이터로 테스트할 때 값이 다르다.
* 카메라 기능, 각종 센서를 테스트하는 것은 시뮬레이터에서 불가능하다.

Reference: <br/>
[실제 디바이스가 없을 경우 개발 환경에서 할 수 있는 것과 없는 것을 설명하시오.](https://hyun083.tistory.com/71) <br/>