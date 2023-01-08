# Swift Collection 관련 프로토콜에 대해 설명하시오

Swift Standard library 에는 컬렉션으로 정의하는 데 유용한 프로토콜을 여럿 가지고 있습니다..

만약 선언한 특정 타입이 아래의 기능을 필요로 한다면 단순히 프로토콜을 구현하는 것만으로 유용한 컬렉션 타입으로 만들 수 있습니다.

## Sequence protocol

요소들에 순서대로 접근하는 기능을 제공합니다. 한 번 방문한 요소는 다시 방문할 수 없습니다(destructive).

순서대로 접근하는 기능을 제공할 수 있기 때문에 for-in loop 에서 사용할 수 있게 됩니다. for-in loop 에서 순서대로 접근하기 위해서는 Iterator 를 반환하는 method(makeIterator())도 추가해야 합니다.

1. 자기 자신이 가진 값이 Iterate 할 수 있음. 자기 자신이 Sequence, IteratorProtocol 을 구현함.
2. Iterate 하는 객체가 따로 존재함. 자기 자신은 Sequence 만 구현하고, makeIterator() 는 IteratorProtocol 을 구현하는 객체를 생성/반환 하도록 함.

```swift
/// 자기 자신이 Sequence, IteratorProtocol 을 구현한 경우
struct Countdown: Sequence, IteratorProtocol {
  var count: Int

  mutating func next() -> Int? {
    if count == 0 {
      return nil // 무한 루프 방지
    } else {
      defer { count -= 1 }
      return count
    }
  }
}

let threeToGo = Countdown(count: 3)
for i in threeToGo {
  print(i)
}
```

```swift
/// 자기 자신은 Sequence, IteratorProtocol 은 별개의 객체가 구현
struct ZeddIterator: IteratorProtocol {
  typealias Element = Int
  var current = 1
    
  mutating func next() -> Element? {
    if current > 10 { return nil } // 무한 루프 방지
    defer { current += 1 }
    return current
  }
}

struct Zedd: Sequence {
  func makeIterator() -> some IteratorProtocol {
    return ZeddIterator()
  }
}
```

## Collection protocol

Sequence 프로토콜의 기능을 확장한 프로토콜입니다. Sequence 프로토콜의 기능과 더불어 아래와 같은 기능을 추가로 갖습니다.

1. 한 번 방문한 요소를 여러 번 방문할 수 있습니다(nondestructive).
2. subscript 문법을 이용하여 특정 인덱스에 접근할 수 있습니다.

Collection 선언부는 다음과 같습니다. 즉, Sequence 를 구현한 객체만이 Collection 구현이 가능합니다.

```swift
public protocol Collection<Element> : Sequence
```

## BidirectionalCollection protocol

Collection 프로토콜의 기능을 확장한 프로토콜입니다.

Collection 프로토콜의 기능과의 차이점으로는 단방향이 아닌 양방향 loop 가 가능하다는 것입니다. LinkedList 와 같은 자료구조에는 불가능합니다.

BidirectionalCollection 선언부는 다음과 같습니다. BidirectionalCollection 의 일부인 indices 는 BidirectionalCollection 이며, SubSequence 도 BidirectionalCollection 임이 보장되어야 합니다.

```swift
public protocol BidirectionalCollection<Element> : Collection where Self.Indices : BidirectionalCollection, Self.SubSequence : BidirectionalCollection
```

## RandomAccessCollection protocol

Collection 프로토콜의 무작위 접근 버전입니다.

어떤 인덱스에 접근하더라도 똑같은 효율(접근 시간)이 보장될 경우 적용 가능합니다. LinkedList 와 같은 자료구조에는 불가능합니다.

해당 타입을 구현하는 데엔 주의점이 한가지 있는데, 접근에 대한 효율성을 보장하기 위해 다음의 작업들 중 하나가 추가로 필요하다는 것입니다.

1. Index 에 해당하는 타입이 Stridable 을 구현한 타입이어야 한다.
2. RandomAccessCollection 프로토콜을 구현한 객체가 index(_:offsetBy:) 를 O(1) 시간 복잡도로 구현해야 한다.
3. RandomAccessCollection 프로토콜을 구현한 객체가 distance(from:to:) 를 O(1) 시간 복잡도로 구현해야 한다.

```swift
protocol RandomAccessCollection<Element> : BidirectionalCollection where Self.Indices : RandomAccessCollection, Self.SubSequence : RandomAccessCollection
```

## Summary

swift 의 Collection 관련 프로토콜들은 연속적인 값들을 처리하기에 최적화된 기능들을 자유롭게 구현할 수 있도록 도와줍니다.

Sequence 프로토콜은 연속적인 값들을 단방향으로 읽을 수 있게 해주고, Collection 프로토콜은 연속적인 값들을 읽는 도중 읽은 값을 다시 읽을 수 있게 해주며, BidirectionalCollection 프로토콜은 단방향이 아닌 단방향으로 읽은 값을 다시 읽을 수 있게 해줍니다.

RandomAccessCollection 프로토콜은 연속된 값의 무작위 접근 기능을 부여하지만, Stridable 한 Index 타입을 갖거나 O(1) 시간 복잡도를 갖는 메소드들이 필요합니다.

위의 프로토콜을 모두 구현한 Swift 의 대표적인 타입은 Array 입니다.

Reference:
* Data Structures & Algorithms in Swift (Fourth Edition): Implementing Practical Data Structures with Swift
* [Apple Docs - Sequence](https://developer.apple.com/documentation/swift/sequence)
* [Swift) Sequence](https://zeddios.tistory.com/1340)
* [Apple Docs - indices](https://developer.apple.com/documentation/swift/collection/indices-swift.property-9kkbf)
* [Apple Docs - SubSequence](https://developer.apple.com/documentation/swift/collection/subsequence)
* [Apple Docs - Collection](https://developer.apple.com/documentation/swift/collection)
* [Apple Docs - BidirectionalCollection](https://developer.apple.com/documentation/swift/bidirectionalcollection)
* [Apple Docs - RandomAccessCollection](https://developer.apple.com/documentation/swift/randomaccesscollection)