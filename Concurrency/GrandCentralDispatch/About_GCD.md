## GCD API 동작 방식과 필요성에 대해 설명하시오.

실행할 작업을 생성한 뒤 DispatchQueue 에 추가하면 GCD 가 작업에 맞는 스레드를 생성하여 실행하고 작업이 종료되면 스레드를 제거한다.

DispatchQueue 는 Queue 이기 때문에 FIFO 방식으로 작동한다.

GCD API 는 두 개의 Queue 를 이용한다. 

* Main Queue 는 Main Thread 를 사용하는 Serial Queue 로 모든 UI 처리는 여기서 담당한다.
* Global Queue 는 Background Thread 를 사용하는 Concurrent Queue 로 처리 우선순위를 정의하는 qos(Quality of service) 파라미터를 통해 병렬적으로 작업을 처리할 수 있게 한다.

경우에 따라서는 작업완료 순서를 정의해야 할 수 있기 때문에 그와 관련된 API 들도 제공되고 있다.

이미지 다운로드를 테이블 뷰와 같이 재사용 뷰에서 보여줘야할 경우, Global Queue 에서 이미지를 다운로드 한 뒤 Main Queue 를 이용해 재사용 뷰에 반영하는 방식으로 처리하면 자연스러운 UI를 만들 수 있다.
