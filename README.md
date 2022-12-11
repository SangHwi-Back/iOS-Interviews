![https://developer.apple.com/wwdc22/images/hero-p3/hero-medium_2x.jpg](https://developer.apple.com/wwdc22/images/hero-p3/hero-medium_2x.jpg)
*출처: https://developer.apple.com/wwdc22/ | Copyright © 2022 Apple Inc. 모든 권리 보유.* 

# iOS-Interviews

> iOS 취업을 위해 정리하고 있는 면접 질문들을 카테고리 별로 정리합니다. 신입/경력 모든 분들께 도움이 되었으면 합니다.

## 질문 및 주제들의 출처

GitHub 에 공유된 질문 모음들과 여러 기업들이 채용공고에 명시한 지원요건을 가장 많이 참고합니다.

여러 기업들이 지원요건을 상세히 쓰고 자주 업데이트하는 그날이 왔으면 좋겠습니다.

__*해당 질문들이나 주제가 불특정 다수에 의해 공유되는 것에 대한 문제가 있을 시 건의해주시면 검토 후 적극 반영할 것입니다.*__

## Table Of Contents

| Name         | Level | Comment                                      |
|--------------|-------|----------------------------------------------|
| CocoaTouch   | *     |                                              |
| Swift        | *     |                                              |
| UIKit        | *     |                                              |
| Architecture | ****  | ReactorKit 추천. MVVM 은 개발자/팀 별 해석이 너무 달라서 보류. |
| LowLevel     | ****  | Swift 기초를 익히는 것을 추천.                         |
| Concurrency  | ***** | Concurrency 라는 단어를 선택한 이유는 폴더 아래에 상세히 설명.    |
| Mathematics  | *     | Swift 를 이용해서 여러 수학적 지식을 활용한 알고리즘 구현을 소개.     |
| ETCs         | ?     | 개발자 채용 시 공통적으로 묻는 질문들을 정리합니다.                |


## CocoaTouch

App LifeCycle 부터 iOS Application 개발환경에 대한 질문들을 모았습니다.

* Cocoa Touch
* iOS App Lifecycle
* iOS Development Environment

## Swift

Swift 기초, Protocol, keyword, 독특한 문법들을 모았습니다.

* FunctionalProgramming
* ObjectOrientedProgramming
* ObserverPattern
* ProtocolOrientedProgramming

## UIKit

UIView 부터 자주 쓰이는 View 들에 대한 질문에 대비합니다.

UIResponder 와 관련된 User Event 들에 대한 질문도 다룹니다.

* AutoLayout
* CollectionViews(e.g. UITableView, UICollectionView...)
* UserInteraction(e.g. UITapGestureRecognizer...)
* ViewProgramming

## Architecture

MVC, RIBs, ReactorKit 등 유용한 아키텍처 중 믿을만한 자료들이 많다고 판단되는 아키텍처에 대해서만 정리합니다.

* MVC
* RIBs
* ReactorKit

## LowLevel

ARC, Compiler 등 Swift 저레벨 지식 관련 질문들을 정리합니다.

* ARC
* MemoryManagement
* Debugger

## Concurrency

비동기 프로그래밍을 사용하기 위한 API, Open-Source 에 대해 정리합니다.

신입 개발자보다는 경력 개발자분들께 더 도움이 될 것 같아서 특정 연산자에 대한 설명과 같이 단순한 개념보다는 난이도가 높은 질문들을 구성해 보았습니다.

* GrandCentralDispatch
* OperationQueue
* RxSwift
* Combine

## Mathematics

일부 기업은 인터뷰 도중 라이브 코딩 테스트를 진행하기도 합니다.

대부분 코딩 테스트 문제를 제공하는 웹사이트의 문제와 비슷한 형태 (예: 별 찍기, 괄호 짝짓기, 룰에 따른 문자열 변환 등) 들이 많은데, 이런 상황에서 많은 도움이 될 것이라고 판단했습니다.

그 외에도 수학적인 지식은 프로그래머에게 여러모로 많은 도움을 (적어도 방해는 되지 않음) 주고 있으므로 참고만 해주시기 바랍니다.

목차는 [Mathematics README](https://github.com/SangHwi-Back/iOS-Interviews/tree/main/Mathematics) 에 나열하도록 하겠습니다.

## ETCs

현재까지 계획된 목록은 다음과 같습니다.

* HTTP Protocol
* Images
* Dynamic Programming
