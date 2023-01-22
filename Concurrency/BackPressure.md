# 리액티브 프로그래밍 도중 발생할 수 있는 BackPressure 현상이 무엇이고 해결방안은 어떤 것이 있는지 설명해주세요.

> "역류" 라는 의미 그대로 너무 많은 이벤트를 구독자가 처리하지 못하는 현상을 일컫습니다. 단순히 사용자의 액션에 반응하는 코드라면 괜찮겠지만, 실시간으로 많은 양의 이벤트를 처리하는 앱을 서비스하는 개발팀에서는 관심을 갖을 수 있겠습니다.

## BackPressure 현상이란?

Reactive Programming 에서 가장 문제가 되는 두 주제는 다음과 같습니다.

* Memory Peak
* Backpressure

여기서 Backpressure 란 Subscriber 와 Publisher(Observable 등 데이터를 전달해주는 주체) 의 데이터 전달 도중 Subscriber 가 이벤트 혹은 값을 소비하는 속도가 Publisher 의 이벤트 혹은 값 전송 속도를 감당하지 못하는 상황을 일컫습니다.

일반적인 상황은 아닙니다. 특히 iOS 개발이 국내에서는 서비스 제공 용도로 많이 사용되는 상황에서 데이터 전달은 유저의 의도에 따라 일어나기 때문에 Publisher 가 데이터를 전달하는 빈도가 상대적으로 많지 않습니다.

가장 많이 발생할 수 있는 상황은 외부 기기와 연동된 상태에서 데이터를 전달받을 때 Reactive Programming 을 사용하는 경우가 있을 것입니다. 짧은 시간동안 어마어마한 양의 데이터가 온다고 예상이 된다면, 특정 시간간격마다 이벤트 혹은 값을 처리하는 방법이 가장 효율적일 것입니다.

이에 대해 RxSwift, Combine 은 각각 다른 해결책을 제시합니다.

* RxSwift = throttle, debounce, buffer, sample, debounce, window 등 stream 을 제어하는 연산자를 이용합니다.
* Combine = Subscriber 프로토콜을 이용해서 Custom Subscriber 를 만든 뒤 내부적으로 stream 을 제어합니다.

## RxSwift 에서의 back pressure

RxSwift 는 기본적으로 Hot, Cold Observable 이 존재합니다. 함수형 프로그래밍에서 말하는 Push 방식과 Pull 방식 여부에 따라 달라지는 Observable 형태이므로 각각의 상황에 따라 대응이 필요합니다.

### Cold Observable

Pull 방식입니다. 이벤트를 시작부터 관찰하는 형태로 마치 이벤트를 처음부터 끌어오는 형태의 Observable 을 말합니다.

Cold Observable 은 Subscriber 가 원하는 때 이벤트를 방출할 수 있는 Observable 을 말합니다. 주로 Subscriber 의 구독에 의해서만 이벤트를 방출하는 Observable 을 Cold Observable 이라고 말합니다.

순환 로직 혹은 네트워크 요청 등에서는 필요한 작업만 방출하여 구독할 수 있는 Cold Observable 은 많은 문제를 일으키지는 않습니다. 하지만 multicast 등을 통해 여러 Observable, Subject 에 영향을 주게 되면 Hot Observable 이 되어 backpressure 현상을 발생시킬 수 있습니다.

### Hot Observable

Push 방식입니다. 이벤트 중간부터 관찰하는 형태로 마치 이벤트가 구독자의 구독 로직을 실행시키는 듯한 형태의 Observable 을 말합니다.

Hot Observable 은 생성과 동시에 이벤트 방출이 바로 진행되는 Observable 을 말합니다. Subscriber 은 일반적으로 Hot Observable 의 이벤트 중 일부만을 전달받습니다.

가장 대표적인 예시로는 마우스 & 키보드 입력, 시스템 관련 이벤트, 주가 정보 등일 것입니다.

이 경우에는 throttle, debounce, buffer, sample, debounce, window 등 연산자를 이용해서 stream 을 제어해야 합니다. 모든 이벤트를 처리하는 것이 아니라, 일정 시간간격을 두고 이벤트를 차례대로 관찰하도록 합니다.

## Combine 에서의 back pressure

구독 자체를 뜻하는 프로토콜인 Subscription 프로토콜은 다음과 같이 구현되어 있습니다.

```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
  func request(_ demand: Subscribers.Demand)
}
```

Subscriber 는 구독을 하게 되면 Subscription 객체를 Publisher 로부터 전달받게 됩니다. Subscription 객체에는 request(_:Subscription.Demand) 함수가 구현되어 있으며, Subscriber 는 request 함수를 호출합니다.

request(_:Subscription.Demand) 함수에는 Subscription.Demand 값을 전달할 수 있게 되어있습니다. Subscription.Demand 는 세 타입의 값이 존재하는데 Subscriber 의 상황에 따라 세 개의 값을 전달할 수 있습니다.

* max(_:Int) = 이번에 전달받은 데이터 이후로 최대 받을 수 있는 갯수의 이벤트 수를 명시할 수 있습니다.
* unlimited = 이번에 전달받은 데이터 이후로 몇개든 이벤트를 전달받을 수 있습니다.
* none = 이번에 전달받은 데이터 이후로는 이벤트를 전달받지 못합니다.

Subscriber 가 현재 데이터가 처리되는 현황을 관찰하면서 다음 데이터 수신을 어떻게 할지 관찰 및 처리할 수 있게 된 것입니다.

References:
* [ReactiveX - backpressure operators](https://reactivex.io/documentation/operators/backpressure.html)
* [\[Swift\] Combine의 Backpressure 처리](https://bonoogi.github.io/posts/2022-03-06-combine-backpressure/)
* Book - Combine: Asynchronous Programming with Swift