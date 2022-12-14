# DispatchQueue 

iOS 앱 개발 시 동시성 프로그래밍을 하기 위해 사용하는 API. Grand Central Dispatch API 를 Swift 로 사용할 수 있도록 하는 저수준 API 에 속합니다..

## 언제 쓸까요?

앱이 동시에 여러 작업을 수행해야 하는 경우 고려해볼 수 있습니다.

동시성 프로그래밍이 가능해진 것은 멀티 스레딩이 가능해진 순간부터 순식간에 여러 개의 스레드에 작업을 저장하고, 스케줄링을 통해 다수의 스레드를 아주 빠른 시간 내에 옮겨다니며 작업을 수행하는 것입니다.

## [용어 정리] Synchronous, Asynchronous, Concurrency

동기, 비동기, 동시성 이라는 세 단어는 정확한 차이를 모르면 실수로 잘못된 의미를 전달할 수 있기 때문에 정확히 짚고 넘어가야 합니다.

* Synchronous(동기) = 특정 작업이 진행되는 Scope 에서 다른 작업의 Scope 를 실행되고 나서 return 이 될 때까지 기다렸다가 return 이 되고 나면 다시 자신의 Scope 로 돌아와서 작업을 이어 수행해 나가는 것을 말합니다.
* Asynchronous(비동기) = 특정 작업이 진행되는 Scope 에서 다른 작업의 Scope 를 실행되고 나서 return 을 기다리지 않고 작업을 이어 수행해 나가는 것을 말합니다.
* Concurrency(동시성) = 다수의 작업을 다수의 스레드에 두어 우선순위에 따라 동시에 처리하거나 의존성에 따라 처리하는 것을 말합니다. 멀티 스레딩에 의해 가능해진 기능입니다.

## [용어 정리] DispatchQueue vs Grand Central Dispatch

이 두 단어도 많은 Article 에서 혼용되고 있습니다. 정확히 말하면 DispatchQueue 는 Grand Central Dispatch 를 개발자들이 사용할 수 있게 만든 저수준 API 입니다.

* Grand Central Dispatch = GCD(Grand Central Dispatch) 라는 C 언어 library 라이브러리가 있습니다. 이를 애플에서 iOS 개발환경에서 사용하기 위해 다시 만들게 됩니다. GCD 는 운영체제가 스레드 관리 및 실행권한을 부여받고 실행할 수 있도록 기능하게 합니다.
* DispatchQueue = GCD 가 제공하는 API 를 이용하는 클래스입니다. DispatchQueue 와 함께 작동하는 클래스들인 DispatchWorkItem, DispatchGroup 등도 모두 GCD API 를 사용하기 위해 존재하는 것이다.

> 즉 GCD 는 GCD API 를 사용하는 라이브러리나 API 등을 지칭하고 있는 것입니다. 

## DispatchQueue

GCD API 를 사용할 때 가장 많이 사용하게 되는 클래스 중 하나입니다. Queue 구조를 따르고 있으며 용도에 따라 여러개의 타입으로 나뉩니다.

DispatchQueue 에 전달하는 데이터는 실행 가능한 Block 혹은 DispatchWorkItem 등 실행 가능한 Block 을 담은 클래스들입니다. DispatchQueue 는 전달받은 Block 들을 Thread 들에 순차적으로 배치한 뒤 Block 을 실행합니다. 

* DispatchQueue.main = UI 를 업데이트하는 Main Thread 를 이용하여 작업을 수행합니다.
* DispatchQueue.global = Multi Threading 환경에서 작업을 나눠 처리하고 싶을 때 사용하는 Queue 입니다.

```swift
// DispatchQueue 의 Background Thread 에서 언제 응답이 올지 모르는 네트워크 요청을 하여
// Main Thread 에 영향을 주지 않기 때문에 사용자 경험에 영향을 주지 않습니다.
DispatchQueue.global(qos: .userInteractive).async {
      
   URLSession.shared.dataTask(with: commonURL) { 
      [weak self] data, response, error in
         
      guard let data,
         error == nil
      else {
         return
      }
         
      self?.updateBackgroundImage(data)
   }
}

func updateBackgroundImage(_ data: Data?) {
   // DispatchQueue 의 Main Thread 에 전달된 Data 를 UIImage 로 바꿔보고
   // UIImage 로 변환이 된다면 업데이트 하는 실행 Block 을 넘겨주고 실행하도록 합니다.
   DispatchQueue.main.async { [weak self] in
   
      guard let image = UIImage(data: data) else { 
         return 
      }
      
      self?.backgroundImageView.image = image
   }
}
```

DispatchQueue.global 은 qos 에 따라 작업의 우선순위가 자동으로 결정됩니다.

1. userInteractive = UI, Event 관련 작업.<br/>
   예시: UI 요소 계산, 에니메이션, 이벤트 핸들링
2. userInitiated = 작업의 실행 및 결과로 사용자가 앱을 일시적으로 사용 못하게 함.<br/>
   예시: 문서를 팝업 형태로 불러옴
3. default = 일반적으로 사용하지 않는 것을 권장한다.<br/>
   (default 는 unspecified -> default -> utility 의 구조를 갖는다.)
4. utility = 사용자가 의도하였지만, 지속적으로 관찰하진 않는 작업에 사용된다.<br/>
   예시: I/O, networking, 지속적인 데이터 inout
5. background = 사용자와의 상호작용이 전혀 없는 작업에 사용된다.<br/>
   예시: Prefetching, backup, 외부 서버와의 동기화 작업
6. unspecified = 이전 버전의 API 와의 호환성을 위해 남겨놓은 값이다.<br/>사용을 권장하지 않음.

지금까지 언급한 DispatchQueue.main 와 DispatchQueue.global 은 static 하게 만들어지는 Queue 들입니다. 하지만, 원한다면 직접 Queue 를 생성할 수도 있습니다.

```swift
let serialQueue = DispatchQueue(label: "com.mycompany.jobInterview")
let concurrentQueue = DispatchQueue(label: "com.mycompany.jobInterview", qos: .userInteractive, attributes: .concurrent)
```

## DispatchWorkItem

DispatchQueue 에 전달할 수 있는 작업의 단위이다. 실행 가능한 코드 Block 을 전달하는 대신 단일 객체로서 정의할 수 있어서 Modulation, Testable 에 용이합니다.

DispatchWorkItem 의 큰 장점은 Item 간 Scheduling 이 가능하다는 점입니다. notify 함수를 통해 다음에 실행될 Item 과 Item 이 전달될 DispatchQueue 를 지정할 수 있습니다.

```swift
var newData: Data?
let fetchForUpdateImageItem = DispatchWorkItem {
   URLSession.shared.dataTask(with: commonURL) { 
      [weak self] data, response, error in
         
      guard let data,
         error == nil
      else {
         return
      }
      
      // newData 변수에 data 를 반영하는 방식도 취할 수 있습니다. 하지만 권장되지 않습니다.
      // 해당 코드가 실행될 스레드 외 다른 스레드에 영향을 미치지 않도록 프로그래밍 하세요.
      // 그렇지 않으면 Race Condition (경쟁 상태) 를 유발시켜 작업자의 의도를 벗어난 결과를 가져올 수 있습니다.
      self?.updateBackgroundImage(data)
   }
}

var uiUpdateItem: (Data) -> DispatchWorkItem = DispatchWorkItem { 
   [weak self] data in
   
   guard let image = UIImage(data: data) else {
      return
   }
   self?.backgroundImageView.image = image
} 

fetchForUpdateImageItem.notify(queue: .main, // execute 로 전달되는 DispatchWorkItem 을 실행할 Queue 정의. 
   execute: uiUpdateItem)
   
DispatchQueue.global(qos: .utility)
   .async(execute: fetchForUpdateImageItem)
```

## DispatchGroup

DispatchGroup 은 DispatchWorkItem 을 작업 단위로 하여 각각 관리합니다. 이 클래스를 이용하여 작업의 의존성을 더욱 효율적으로 관리할 수 있습니다. 예를 들어

* 어떤 비즈니스 로직을 동시적으로 관리하기 위한 Model 클래스가 필요합니다.
* Model 내에 DispatchGroup 객체를 저장해놓고 다른 객체들이 비동기적으로 실행될 코드를 Model 에게 전달합니다.
* Model 은 코드들을 DispatchGroup 에 넣어 관리합니다.

DispatchGroup 의 가장 중요한 점은 **작업 취소가 가능**하다는 것입니다. 의존성을 부여한 대로 작업을 진행하고 있는 상태에서 어떤 상태값 등을 만났을 때 개발자의 의도대로 작업을 취소시킬 수 있는 것입니다.

## 전문가의 조언

아주 유명한 도서 중 하나인 'CleanCode' 라는 책에 이런 내용을 일부 발췌하였습니다.

> 다음은 동시성과 관련한 일반적인 미신과 오해다.
> 
> * 동시성은 항상 성능을 높여준다.
> * 동시성을 구현해도 설계는 변하지 않는다.
> * 웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.
> 
> 반대로 다음은 동시성과 관련된 타당한 생각 몇 가지다.
> 
> * 동시성은 다소 부하를 유발한다.
> * 동시성은 복잡하다.
> * 일반적으로 동시성 버그는 재현하기 어렵다.
> * 동시성을 구현하려면 흔히 근본적인 설계 전략을 재고해야 한다.

즉, 동시성은 앱의 성능을 `분명히` 향상시켜줄 것이라는 근거가 부족한 믿음을 가지게 할 수도 있다는 것입니다.

동시성 프로그래밍 자체를 사용하지 않으면 동시성 프로그래밍에서 흔히 문제로 지적되는 다음의 사항들에서 자유로워질 수 있습니다. 이것 또한 동시성 프로그래밍이 앱의 성능을 `분명히` 향상시키는 것만큼은 성능을 향상시켜 줍니다.

* Race Condition (경쟁 상태)
* DeadLock (교착 상태)
* Priority Inversion (우선 순위 반전)

즉 동시성 프로그래밍이 필요한 무거운 작업이 있을 때는 화면을 잠그거나, 화면을 불러오지 않은 상태에서 무거운 네트워크 작업을 진행한 뒤 화면을 불러오는 등 제 3의 전략도 고려해볼 만 합니다.  

## Summary

GCD(Grand Central Dispatch) 는 애플의 멀티 스레딩 환경을 위해 C 언어에 있던 동명의 라이브러리를 자체 개발한 라이브러리 입니다. 해당 라이브러리들은 API 를 제공하는데, 이를 쉽게 사용하기 위해 DispatchQueue, DispatchWorkItem 등의 클래스를 Cocoa Touch 에서 제공하고 있습니다.

단순히 static 하게 존재하는 DispatchQueue 들을 나누어 사용해도 되며, 클래스 초기화를 통해 직접 Queue 를 생성할 수도 있습니다. DispatchQueue 는 크게 main, global 로 나뉘는데 이는 Main Thread 를 사용하느냐 Background Thread 를 사용하느냐의 차이입니다.

Queue 를 선택할 땐 화면을 업데이트 하는 코드는 Main Thread 에서 실행해야 한다는 점을 주의해야 합니다.

## Tips

동시성 프로그래밍과 그와 관련된 클래스들에 대한 질문은 꼬리질문을 유도하기에 아주 좋습니다. 이에 대해 여러 생각을 해 볼 필요가 있습니다.

예를 들어 _**포트폴리오로 제출한 GitHub 소스코드에 DispatchQueue 를 많이 사용하셨는데 어떤 기준에서 사용하게 되셨나요?**_ 라는 질문은 DispatchQueue 를 사용하는 본인만의 전략이 있는지? 혹은 DispatchQueue 말고 OperationQueue 도 있는데 DispatchQueue 를 선택한 이유는 무엇인지? 등에 집중하여 답변해주시면 좋을 것 같습니다.

물론 OperationQueue, RxSwift, Combine 등을 모르는 상태라면 "제가 가장 자신있는 동시성 프로그래밍 클래스를 사용했습니다." 라고 대답해도 좋습니다만 "~ 와 같이 사용했더니 좋았습니다." 등의 개인적인 경험이 있다면 좋을 것 같습니다.

가장 좋은 방법은 OperationQueue 와 DispatchQueue 의 차이점과 어떤 판단 아래 해당 클래스를 사용했는지 간단히 설명하는 것이 좋다고 생각합니다. 어차피 굉장히 많은 얘기를 한꺼번에 하여도 면접관분의 관심사를 알 수는 없으니 꼬리 질문을 하시는 것을 들어보고 추가설명을 해도 늦지 않기 때문입니다.

모든 Tip 을 통틀어 가장 중요한 것은 **전문가의 조언** 에서도 언급했듯이 "사용하지 않아도 해결할 수 있는 문제임에도 동시성 프로그래밍을 사용해야 한다고 판단" 한 정확한 근거가 있다면 더할나위 없이 좋다는 것입니다.

Reference:
* [https://velog.io/@sanghwi_back/Swift-동시성-프로그래밍-2-DispatchQueue#grand-central-dispatch](https://velog.io/@sanghwi_back/Swift-%EB%8F%99%EC%8B%9C%EC%84%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-2-DispatchQueue#grand-central-dispatch)
* [도서] Concurrency by Tutorials(2nd Edition): Multithreading in Swift with GCD and Operations - raywenderlich Tutorial Team, Scott Grosch 지음