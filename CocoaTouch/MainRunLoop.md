# iOS 애플리케이션의 필수 동작인 RunLoop 와 Main Run Loop 에 대해 설명하시오

## RunLoop

![Structure of a run loop and its sources](../_Images/MainRunLoop_Image1.jpg)

Run Loop 는 이벤트 처리를 위한 실행 Block 을 실행시키는 루프입니다. 실행 Block 은 Run Loop 에 의해 적절한 Thread 로 전달됩니다.

이 중 특별히 Main 함수에 의해 실행되어 Main 스레드의 작업에 대한 실행 Block 을 관리하는 루프는 Main Run Loop 라고 부릅니다.

Run Loop 의 가장 중요한 장점은 수행해야 할 작업이 있을 경우는 스레드를 활성화하고 그렇지 않을 경우는 스레드를 비활성화시키는 것입니다.

Cocoa / Core Foundation 프레임워크 모두 Run Loop 에서의 스레드 동작을 정의하기 위한 객체들이 제공됩니다.

Run Loop 가 입력 받는 이벤트는 2 가지로 나뉩니다. Run Loop 는 이벤트를 입력받으면 전달받은 실행 Block 을 실행합니다. 가장 큰 차이는 동기/비동기 여부입니다.
* Input Sources = 다른 스레드나 앱으로부터의 전달받은 비동기 이벤트를 말합니다.
* Timer Sources = 일정한 시간마다 실행되는 동기 이벤트들을 말합니다.

실제 Run Loop 클래스를 사용해야 하는 상황은 부차적인 스레드를 만들어야 할 경우 입니다. 

Thread 에 알맞은 Run Loop 가 없다면 자동으로 생성되기 때문에 Main 스레드에서 실행되어야 할 작업이 아니라면 `RunLoop.current` 로 Run Loop 에 접근하도록 합니다.

## Run Loop 를 이용한 작업 실행

Run Loop 의 실행은 자동으로 되지 않으므로 직접 `run()` 메소드를 이용해 실행해야 합니다.

```swift
DispatchQueue.global().async {
  let isRunning = true
  
  // 1. 이벤트를 발생시킨다.
  Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
    // Do some stuff.
  }
  
  // 2. Run Loop 객체의 run() 을 호출한다.
  RunLoop.current.run()
  
  // 3. 이벤트는 발생하였지만 런 루프에 포함되지 않았으므로 처리되지 않음.
  Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
    // Do some stuff.
  }
}
```

앞에서 언급했듯이 Input Sources 는 비동기로 처리되는 이벤트로서 스레드에 전달되게 됩니다. Timer Source 는 이와 반대로 동기로 처리되는 이벤트로서 스레드에 전달되게 됩니다.

바로 위의 코드처럼 GCD 를 이용해서 작업을 처리해야 할 경우 잘못된 작업 Queue 사용으로 인해 의도한 작업이 실행되지 않을 수 있습니다. [DispatchQueue(GCD)](../Concurrency/GrandCentralDispatch/DispatchQueue.md)

```swift
// Synchronous 한 Timer 작업 예시

// 1. Concurrent Queue 즉, 비동기 큐에서 Timer 작업 실행.
DispatchQueue.global().async {
  Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
    // Do some stuff.
  }
}
// 결과 : 실행 안됨. 직접 async 클로저 안에서 Run Loop 실행을 해줘야 함.

// 2. 동기화된 환경에서 Timer 작업 실행
Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
  // Do some stuff.
}
// 결과 : 동기화된 환경 안에서는 자동으로 실행 됨. 직접 Run Loop 실행을 관리할 필요가 없음.
```

## Main Run Loop 실행 Matrix

Main 함수에 의해 실행되는 Run Loop 입니다. Main Thread 에 의해 실행되어야 할 작업을 처리해야 하기 때문에 종료되지 않습니다. 앱의 실행과 라이프사이클을 함께 합니다.

* 앱에서 이벤트(터치 등)가 전달되면 iOS 운영체제 커널에서는 해당하는 커널 포트를 통해 이벤트 Queue 로 이벤트를 저장해 놓게 된다.
* Queue 에 저장되었으므로, 순차적으로 각 앱의 Main Run Loop 로 이동하게 된다.
* Main Run Loop 는 앱의 개체들을 불러와서 핵심 개체들(Foundation, UIKit 등)을 이용해 화면 렌더링을 지시하게 된다.
* iOS 운영체제 커널은 지시된 화면 렌더링을 앱에 지시하게 된다.

Main Run Loop 혹은 Main Thread Run Loop 는 앱의 핵심 요소인 AppDelegate, SceneDelegate, ViewController 뿐만 아니라 작성된 코드들에도 접근이 가능합니다.

References:

* [Apple Documentations - Run Loops](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
* [\[iOS - swift\] Run Loops (런 루프, Thread 프로그래밍, global queue 에서 Timer 동작 방법) ](https://ios-development.tistory.com/515)
* [iOS - RunLoop 란?](https://minosaekki.tistory.com/54)
