# iOS 애플리케이션의 필수 동작인 RunLoop 와 Main Run Loop 에 대해 설명하시오

## RunLoop

![Structure of a run loop and its sources](../_Images/MainRunLoop_Image1.jpg)

Run Loop 는 이벤트 처리 루프입니다. 수행할 작업을 스케줄링 하고 이벤트 입력을 받는 작업을 담당합니다. Run Loop 의 주 목적은 수행해야 할 작업이 있을 경우는 스레드를 활성화하고 그렇지 않을 경우는 스레드를 비활성화시키는 것입니다.

Cocoa / Core Foundation 프레임워크 모두 Run Loop 에서의 스레드 동작을 정의하기 위한 객체들이 제공됩니다. 앱의 프레임워크는 앱이 실행될 때 Run Loop 를 메인 스레드에서 자동으로 실행되도록 합니다.

Run Loop 가 입력 받는 이벤트는 2 가지로 나뉩니다. 가장 큰 차이는 동기/비동기 여부입니다.
* Input Sources = 비동기 이벤트를 전달합니다. 주로 다른 스레드나 앱으로부터 전달받은 메시지(작업, 함수)입니다.
* Timer Sources = 동기 이벤트를 전달합니다. 스케줄 된 시간이나 반복적인 시간마다 실행되는 이벤트들을 말합니다.

Run Loop 가 사용되어야 하는 상황은 부차적인 스레드를 만들어야 할 경우 입니다. main 스레드에 의해 관리되는 Run Loop 는 실제로 다룰 경우 문제가 되기 때문에 main 함수에서 자동으로 실행됩니다. 만약 장시간 실행하는 작업이나 미리 실행되어야 하는 작업 등이 아닌 사용자와 실시간으로 피드백이 필요한 작업이라면 Run Loop 를 사용할 수 있습니다.

```swift
DispatchQueue.global().async {
  let isRunning = true
  
  // 1. 이벤트를 발생시킨다.
  Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in 
    // Do some stuff.
  }
  
  // 2. Run Loop 객체의 run() 을 호출한다.
  RunLoop.current.run()
}
```

## Main-RunLoop

Main 함수에 의해 실행되는 Run Loop 입니다.

* 앱에서 이벤트(터치 등)가 전달되면 iOS 운영체제 커널에서는 해당하는 커널 포트를 통해 이벤트 Queue 로 이벤트를 저장해 놓게 된다.
* Queue 에 저장되었으므로, 순차적으로 각 앱의 Main Run Loop 로 이동하게 된다.
* Main Run Loop 는 앱의 개체들을 불러와서 핵심 개체들(Foundation, UIKit 등)을 이용해 화면 렌더링을 지시하게 된다.
* iOS 운영체제 커널은 지시된 화면 렌더링을 앱에 지시하게 된다.

Main Run Loop 혹은 Main Thread Run Loop 는 앱의 핵심 요소인 AppDelegate, SceneDelegate, ViewController 뿐만 아니라 작성된 코드들에도 접근이 가능합니다.

References:

* [Apple Documentations - Run Loops](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
* [\[iOS - swift\] Run Loops (런 루프, Thread 프로그래밍, global queue작 에서 Timer 동작 방법) ](https://ios-development.tistory.com/515)
* [iOS - RunLoop 란?](https://minosaekki.tistory.com/54)
