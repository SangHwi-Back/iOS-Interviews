GCD API 동작 방식과 필요성에 대해 설명하시오.

실행할 작업을 생성한 뒤 DispatchQueue에 추가하면 GCD가 작업에 맞는 스레드를 생성하여 실행하고 작업이 종료되면 스레드를 제거한다.

Queue 이기 때문에 FIFO방식으로 작동한다.
GCD API는 두 개의 Queue를 이용한다. Main Queue는 Main Thread를 사용하는 Serial Queue로 모든 UI처리는 여기서 담당한다. Global Queue는 Background Thread 를 사용하는 Concurrent Queue로 처리 우선순위를 정의하는 qos(Quality of service) 파라미터를 통해 병렬적으로 작업을 처리할 수 있게 한다.
작업완료 순서는 정할 수 없어도 우선적으로 처리 가능하게는 할 수 있다.

이미지 다운로드를 테이블 뷰와 같이 재사용 뷰에서 보여줘야할 경우, Global Queue에서 이미지를 다운로드 한 뒤 Main Queue를 이용해 재사용 뷰에 반영하는 방식으로 처리하면 자연스러운 UI를 만들 수 있다.
