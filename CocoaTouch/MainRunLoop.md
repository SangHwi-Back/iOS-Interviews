# iOS 애플리케이션의 필수 동작인 RunLoop 와 Main Run Loop 에 대해 설명하시오

## RunLoop

![Structure of a run loop and its sources](../_Images/MainRunLoop_Image1.jpg)

`Run Loop` 는 전달된 이벤트의 Event Handler 를 실행시키기 위한 루프입니다. Event Handler 는 등록된 Run Loop Observer 에게 적절한 Notification 을 수신하면 Thread 로 전달되고 실행됩니다.

이 중 특별히 Main 함수에 의해 실행되어 Main 스레드에서 실행되어야 하는 Event 를 실행하는 루프는 `Main Run Loop` 라고 부릅니다.

Run Loop 의 가장 중요한 장점은 수행해야 할 작업이 있을 경우는 스레드를 활성화하고 그렇지 않을 경우는 스레드를 비활성화시키는 것입니다.

Cocoa / Core Foundation 프레임워크 모두 Run Loop 에서의 스레드 동작을 정의하기 위한 객체들이 제공됩니다.

Run Loop 가 입력 받는 이벤트는 2 가지로 나뉩니다. Run Loop 는 이벤트를 입력받으면 전달받은 실행 Block 을 실행합니다. 가장 큰 차이는 동기/비동기 여부입니다.

* Input Sources = 다른 스레드나 앱으로부터의 전달받은 비동기 이벤트를 말합니다. run(until:) 메소드를 호출하여 종료시점을 정할 수 있습니다.
* Timer Sources = 일정한 시간마다 실행되는 동기 이벤트들을 말합니다. 

실제 Run Loop 클래스는 부차적인 스레드를 만들어야 할 경우 사용됩니다. 앱에서 부차적인 스레드가 필요한 경우의 예시는 다음과 같습니다.

* port 또는 custom input source 를 사용하여 다른 스레드와 통신해야 되는 경우
* 스레드에서 Timer 를 사용해야 하는 경우
* Cocoa 의 objc 런타임을 사용하는 performSelector 를 사용해야 하는 경우
* 일정한 시간마다 반복적인 작업을 위해 스레드를 유지해야 하는 경우

## Run Loop Mode

Run Loop Mode 는 관찰받을 Event Source 들과 관찰하는 Run Loop Observer 들 모두를 가리킵니다. (해당 표현은 굉장히 포괄적인 의미로 Run Loop Mode 를 설명할 수 없지만, [공식문서 Run Loop Modes](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html) 의 언급이므로 서술하였습니다.)

정확히 얘기하자면 Run Loop 가 실행되는 주기 동안 Mode 라는 값을 통해 어떤 Event Source 가 어떤 Observer 에 의해 모니터링 되다가 실행되어야 하는지 특정할 수 있는 것입니다.

이는 반대로 어떤 Observer 가 특정 Event Source 는 모니터링 하지 않도록 필터링하게 만들 수 있다는 것이기도 합니다.

참고로 Mode 는 이벤트를 이벤트 타입이 아닌 이벤트 게시자로 식별합니다. 예를 들어 '마우스 클릭', '키보드 입력' 등으로만 인식하는 것입니다.

Mode 는 지정된 String 을 통해 식별할 수 있습니다. 

| Name                                                                     | Description                                                                                                                         |
|--------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| NSDefaultRunLoopMode (Cocoa)<br/>kCFRunLoopDefaultMode (Core Foundation) | 기본모드. 대부분의 작업에 사용됨.                                                                                                                 |
| NSConnectionReplyMode (Cocoa)                                            | NSConnection 객체를 위한 모드.                                                                                                             |
| NSModalPanelRunLoopMode (Cocoa)                                          | Modal Panel 을 감지하는 모드.                                                                                                              |
| NSEventTrackingRunLoopMode (Cocoa)                                       | Mouse 드래그와 같이 추적이 필요한 이벤트 발생 시 다른 이벤트들은 차단하고 싶을 때 사용하는 모드.                                                                          |
| NSRunLoopCommonModes (Cocoa)<br/>kCFRunLoopCommonModes (Core Foundation) | 자유롭게 설정 가능한 모드. Cocoa 의 경우 Default/Modal/EventTracking 모드가 관찰하는 이벤트, Core Foundation 의 경우 Default 모드가 관찰하는 이벤트를 특정 옵저버와 짝지을 수 있습니다. |

## Run Loop 생성

애플리케이션은 Run Loop 객체를 명시적으로 생성하거나 관리하지 않습니다.

Thread 에 알맞은 Run Loop 가 없다면 자동으로 생성됩니다. Main 스레드에서 실행되어야 할 작업이 아니라면 `RunLoop.current` 로 Run Loop 에 접근하도록 합니다.

Run Loop 의 의사코드는 다음과 같습니다.

```swift
// Message 를 runloop 작업 큐에 넣고 signal 보냄
func postMessage(runloop, message) {
  runloop.queue.pushBack(message)
  runloop.signal()
}

// runloop 가 실행되면 message 를 무한히 기다리다가 message 가 들어오면 적절한 스레드로 message 를 dispatch 한다.
func run(runloop) {
  do {
    runloop.wait()
    message = runloop.queue.popFront()
    dispatch(message)
  } while(true)
}
```

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

## Run Loop 의 문제점과 해결방안

### Timer Tolerance

만약 Timer 에 대한 Event Handler 가 등록된 Run Loop 에 굉장히 많은 Input sources 가 들어왔다고 가정해봅니다. Run Loop 는 Queue 에 이벤트를 저장하기 때문에 순차적으로 작업을 처리합니다.

`Massive Input Sources + Repeat Timer`

이렇게 될 경우 기기에 따라서 연산속도에 차이가 발생할 수 있기 때문에 기존에 의도한 Timer 의 Event Handler 실행주기에서 짧게나마 지연이 발생하게 됩니다. 이를 `Timer Tolerance` 라고 합니다. 

만약 쌓인 지연시간이 Timer 의 주기(Interval) 을 초과하면 '넘어간 주기 + 현재 주기' 동안 Event Handler 는 한번만 실행됩니다. 원래 의도한 실행횟수(2번)과 다릅니다.

이는 2 가지 방법으로 해결이 가능합니다.

1. Timer class 의 tolerance 프로퍼티에 적절한 값을 반영합니다.
   - 이 프로퍼티는 정해진 주기 이후 발생한 신호를 언제까지 허용해줄 것인지를 설정하는 것입니다.
   - 기본값은 0(TimerInterval 타입) 입니다.
2. 다른 RunLoop 에 Timer 를 추가한다.

```swift
let timer = Timer(timeInterval: 1.0, target: self, selector: #selector(fireTimer), repeats: true)
RunLoop.current.add(timer, forMode: .common)
```

## iOS 에서의 Main Run Loop 실행 Matrix

Main 함수에 의해 실행되는 Run Loop 입니다. Main Thread 에 의해 실행되어야 할 작업을 처리해야 하기 때문에 종료되지 않습니다. 애플리케이션의 실행과 라이프사이클을 함께 합니다.

* 애플리케이션에 Event(제스처 등)가 전달되면 iOS 운영체제 커널에서는 해당하는 커널 포트를 통해 이벤트 Queue 로 이벤트를 저장합니다.
* Queue 에 저장된 Event 는 순차적으로 각 애플리케이션의 Main Run Loop 로 이동합니다.
* Main Run Loop 는 애플리케이션의 객체들 중 핵심 개체들(Foundation, UIKit 등)을 이용해 화면 렌더링을 지시합니다.
* iOS 운영체제 커널은 지시된 화면 렌더링을 애플리케이션에 지시합니다.

Main Run Loop 혹은 Main Thread Run Loop 는 애플리케이션의 핵심 요소인 AppDelegate, SceneDelegate, ViewController 뿐만 아니라 작성된 코드들에도 접근이 가능합니다.

References:

* [Apple Documentations - Run Loops](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
* [\[iOS - swift\] Run Loops (런 루프, Thread 프로그래밍, global queue 에서 Timer 동작 방법) ](https://ios-development.tistory.com/515)
* [iOS - RunLoop 란?](https://minosaekki.tistory.com/54)
* [UIKit rendering - The run loop](https://fabernovel.github.io/2021-01-04/uikit-rendering-part-3)
* [Timer](https://velog.io/@wansook0316/Timer)