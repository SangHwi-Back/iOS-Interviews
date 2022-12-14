## NSOperationQueue 와 DispatchQueue, OperationQueue 의 차이점을 설명하시오.

GrandCentralDispatch(C 기반의 저레벨 API) 를 지칭할 때 가장 많이 언급되는 DispatchQueue, DispatchQueue 에 비해 고레벨 API 로 평가받는 OperationQueue, Objective-C 기반의 NSOperation 은 모두 iOS 개발 환경에서 비동기/동시성 프로그래밍이 가능하도록 한다.

> 참고 : 비동기는 함수 등을 실행한 뒤 return 때까지 기다리는 동기화된 작업의 반대. return 을 기다리지 않고 바로 다음 작업을 수행한다.
> 
> 참고 : 동시성 프로그래밍이란 프로그램이 여러 개의 작업에 멀티 스레딩 환경에서 동시에 작업을 수행하도록 하는 프로그래밍 하는 것.

---

NSOperationQueue 는 NSOperation 객체를 기본 단위로 하는 OperationQueue 를 다루는 방식을 말한다.

NSOperation 은 OperationQueue 에 추가되어 우선순위 및 대기상태에 따라 작업을 완료시키고 제거한다.

작업과 관련된 재개, 취소, 중지와 관련된 API 도 제공된다.

---

OperationQueue 는 모든 작업이 끝난 상태에서 중지되지 않으면 Memory Leak 이 발생할 수 있으니 주의해야 한다.

사용방법이 NSOperationQueue, DispatchQueue 보다 훨씬 간단하다는 점, 그러나 오버헤드가 발생하기 쉽다는 점, KVO 를 통한 작업이 가능하다는 점 등이 특이사항이다.

---

DispatchQueue 는 앱의 Main Thread 혹은 Global Thread 의 작업을 Async 하게 관리하는 객체이다.

Main Thread 에서 작업을 Sync 로 실행하면 DeadLock 이 발생할 수 있다.
