![https://developer.apple.com/wwdc22/images/hero-p3/hero-medium_2x.jpg](https://developer.apple.com/wwdc22/images/hero-p3/hero-medium_2x.jpg)
*출처: https://developer.apple.com/wwdc22/ | Copyright © 2022 Apple Inc. 모든 권리 보유.* 

# iOS-Interviews

> iOS 취업을 위해 정리하고 있는 면접 질문들을 카테고리 별로 정리합니다. 신입/경력 모든 분들께 도움이 되었으면 합니다.

## Table Of Contents

| Name         | Level  | Comment                                      |
|--------------|--------|----------------------------------------------|
| Swift        | **     |                                              |
| UIKit        | **     |                                              |
| Architecture | *****  | ReactorKit 추천. MVVM 은 개발자/팀 별 해석이 너무 달라서 보류. |
| LowLevel     | ****   | Swift 기초를 익히는 것을 추천.                         |
| Concurrency  | *****  | Concurrency 라는 단어를 선택한 이유는 폴더 아래에 상세히 설명.    |

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

## Swift

Swift 기초, Protocol, keyword, 독특한 문법

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