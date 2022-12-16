## GCD API 동작 방식과 필요성에 대해 설명하시오.

GCD 는 멀티 스레딩 환경에서 작업 스케줄링을 위해 만들어진 라이브러리입니다. GCD 는 API 를 제공함으로써 실행할 작업들을 적절한 스레드에 배정한 뒤 실행되도록 합니다.

GCD 는 작업에 따라 2 가지 종류의 Queue 를 제공합니다.

* Main Queue 는 Main Thread 에 실행 Block 을 전달하기 위해 사용하는 Serial Queue 로 모든 UI 처리 관련 Block 은 이 Queue 에 전달해야 합니다.
* Global Queue 는 Background Thread 에 실행 Block 을 전달하기 위해 사용하는 Concurrent Queue 입니다. 처리 우선순위를 정의하는 qos(Quality of service) 파라미터를 통해 병렬적으로 작업을 처리할 수 있기 때문에 무거운 작업들을 동시에 처리하기에도 용이합니다.

GCD 의 활용방안은 무궁무진 합니다. 예를 들어 이미지를 다운로드 하여 UITableView 에 보여주어야 할 경우 다음의 작업이 필요합니다.
* Cell 의 위치와 순서, 내부 UI Layout, Display 등은 Main Queue 를 통해 Main Thread 가 작업을 수행하도록 합니다.
* Image 를 다운로드하는 작업 자체는 네트워크 상황에 따라 오래 걸릴 수 있기 때문에 Global Queue 를 통해서 Background Thread 가 작업을 수행하도록 합니다.
* Image 를 다운로드한 뒤에 이미지를 반영하는 작업은 UI 를 업데이트 하는 작업이므로 Main Queue 를 통해 Main Thread 에서 작업을 수행하도록 합니다.

더 나은 사용자 경험, 효율적인 작업 처리 등을 위해 GCD 를 사용하게 되는 것입니다.