실제 디바이스가 없을 경우 개발 환경에서 할 수 있는 것과 없는 것을 설명하시오.

실제 디바이스가 있건 없건 기본적인 iOS의 기능이나 레이아웃 테스트 등은 가능하다.
애니메이션, 제스쳐, 화면의 해상도와 색상 등은 재현 가능하나 실제 디바이스와 다를 수 있다.
또한 시뮬레이터는 macOS에서 iOS환경을 재현한 것이기 떄문에 foreground/background 환경에 따른 작업을 100% 구현하기 어렵다. 이는 메모리 점유량에서도 마찬가지인데, 실제보다 더 많은 메모리를 점유하는 경우가 일반적이다.
푸시 알림, 카메라 등의 기능을 테스트하는 것은 시뮬레이터에서 불가능하다.