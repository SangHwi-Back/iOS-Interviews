NSOperationQueue 와 GCD Queue 의 차이점을 설명하시오.

둘 다 iOS에서 멀티스레딩을 하는 방법. GCD는 C 기반의 저레벨 API, NSOperation은 Obj-C 기반의 고레벨 API이다.

NSOperationQueue는 NSOperation객체를 기본 단위로 하는 Operation-Queue를 다루는 방식을 말한다. NSOperation은 Operation-Queue에 추가되어 우선순위 및 대기상태에 따라 작업이 시작/완료될 시키고 제거한다(재개, 취소, 중지를 제공하기는 하지만 구현이 어려움). Operation-Queue는 모든 작업이 끝난 상태에서 중지되지 않으면 Memory Leak이 발생할 수 있으니 주의해야 한다. 오버헤드가 발생하기 쉬우며, KVO를 통한 작업이 가능하다.

GCD Queue는 앱의 메인 스레드(큐) 혹은 백그라운드 스레드(큐)의 작업을 동기적 혹은 비동기적으로 관리하는 객체이다. 메인 스레드(큐)에서 작업을 동기적으로 실행하면 교착상태가 발생할 수 있다.
