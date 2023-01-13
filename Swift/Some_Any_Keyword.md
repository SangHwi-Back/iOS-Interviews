# some, any 키워드에 대해 아시는대로 말씀해주세요.

> [WWDC 22 - Embrace Swift generics](https://developer.apple.com/videos/play/wwdc2022/110352/) 의 내용을 많이 참고하였습니다.<br/>
> 많은 내용이 포함되어 있으니 Summary 만 읽으실 수 있도록 작성하였습니다.

some 키워드와 any 키워드는 Swift 추상화를 구현할 때 유용한 키워드입니다.

여러 개의 반복적인 boiler-plate 코드를 만드는 것은 관리적인 측면이나 작성하는 측면에서 모두 비용이 많이 드는 일이기 때문에 추상화는 아주 유용합니다.

추상화된 타입을 통해 하나의 코드가 여러 방식으로 작동하는 것을 '다형성' 이라고 합니다.

추상화를 통한 다형성을 가진 코드를 작성하는 방법은 세 가지가 있습니다.

* Overriding Morphism - 오버라이딩을 통한 다형성
* Sub type Morphism - 하위 타입을 이용한 다형성
* Parametric type Morphism - 매개변수를 이용한 다형성

이 중 some, any 키워드는 `Parametric type Morphism` 을 구현하는 데 아주 유용합니다.<br/>(Overriding, Sub type 다형성은 상속을 기반으로 합니다)

some, any 키워드가 구현하는 Opaque 타입에 대해 알아보는 것부터 시작하도록 하겠습니다.

some/any 중에는 some 을 우선적으로 고려해볼 수 있습니다.<br/>마치 변경여부가 확실하지 않은 변수를 let 으로 선언하는 것이 권장되는 것과 같습니다.

## Opaque, Underlying 타입

구체 타입의 place holder 를 나타내는 타입을 Opaque(불투명) 타입이라고 말합니다. 대체되는 특정 구체 타입을 Underlying(기본) 타입이라고 말합니다.

Opaque 타입은 공통되는 특징을 가진 어떠한 타입이라고 할 수 있습니다. 대표적으로는 여러분들이 만든 protocol 을 구현한 여러 개의 구조체나 클래스들이 구현했다면, protocol 을 구현한 Opaque 타입으로 표현될 수 있는 것입니다.

하지만 구현한 구조체/클래스 자체의 타입은 Opaque 타입이 아닌 Underlying 타입이라고 할 수 있습니다.

( Opaque Type -> *Implementation* -> Underlying Type )

결과적으로 Opaque 타입을 사용할 경우 값은 Underlying 타입으로 한정되어 Underlying 타입을 얻는 것을 보장받을 수 있습니다.

```swift
// Opaque 타입은 Animal, Underlying 타입은 Chicken 입니다.
// Chicken 은 Animal 을 구현한 Underlying 타입입니다.
let chicken: some Animal = Chicken()

// Error!! 이미 chicken 변수는 Underlying 타입으로 한정되었으므로 다른 Underlying 타입을 참조할 수 없습니다.
chicken = Horse()
```

Opaque 타입 추론(Opaque 타입을 Underlying 타입으로 추론)은 호출하는 부분, 반환하는 부분에서 가능합니다.

```swift
// Infer at call
func feed(_ animal: some Animal) { ... }

feed(Horse())
feed(Tiger())

// Infer at return
func checkAnimal(at farm: Farm) -> some Animal {
  Chicken()
}
```

여기서 주의해야 할 사항은 some 키워드를 사용한 타입은 Underlying 타입으로 `한정` 된다는 것입니다.<br/>다음과 같은 상황에 대해서는 컴파일 에러가 발생합니다.

```swift
// Chicken, Cow 중 하나만 return 해야 함
func checkAnimal(at farm: Farm) -> some Animal {
  if farm.location == Location.north {
    return Chicken()
  } else {
    return Cow()
  }
}
```

> 참고 : SwiftUI 에서 `var body: some View` 의 경우 여러가지 뷰를 리턴할 수 있는데, return 키워드가 없고 여러 타입의 View 를 반환할 수 있습니다. @ViewBuilder 가 제공하는 DSL 기능의 하나로, 각 조건에 따른 Underlying 타입을 사용하도록 빌드할 수 있습니다.

이런 추상화는 모듈이나 코드를 작성함에 있어 유리한데, 호출하는 쪽에서 기본 타입(Underlying Type) 을 private 하게 소유할 수 있기 때문입니다.

## some 키워드

some 키워드는 적합하게 프로토콜을 구현한 Opaque 타입을 선언할 때 사용됩니다.

Swift 5.7 부터 some 키워드는 property, parameter, return 타입을 표현할 때 사용할 수 있습니다.

some 키워드 내에서 사용되는 Underlying 타입은 하나로 고정됩니다.

```swift
// [some ProtocolName] => X
func checkSum(_ numbers: [some ProtocolName]) -> some CustomResultType { ... }

// [any ProtocolName] => O
func checkSum(_ numbers: [any ProtocolName]) -> some CustomResultType { ... }
```

## any 키워드

any 키워드는 type erasure 기능을 구현하기 위해 사용하는 키워드입니다. 이를 통해 any 키워드가 포함된 타입은 동적 타입이 됩니다.

any 키워드는 some 키워드에 의해 고정된 Underlying 타입의 한계를 극복합니다.<br/>예를 들어 Animal 프로토콜을 구현한 객체를 저장하는 배열을 선언한다고 해보겠습니다.

`[some Animal]`

이 코드는 컴파일 에러를 일으킵니다.

`[some Animal]` 은 `Cow`, `Horse`, `Cat` 등 모든 객체를 포함시키고 싶지만 위에서 말한대로 하나의 Underlying 타입으로 고정됩니다.

그러므로 아래와 같이 표현해야 합니다.

`[any Animal]`

즉 any 키워드는 특정 Opaque 타입이 모든 종류의 Underlying 타입을 표현할 수 있다고 선언하는 것과 같습니다.

그리고 Underlying 타입이 런타임마다 달라질 수 있다는 것도 선언하는 것과 같습니다.

## 왜 some, any 키워드를 나누나요??

그렇다면 왜 some, any 가 나뉘어서 존재할까요? any 만 쓰도록 해주면 안되는 걸까요?

some, any 키워드 모두 객체에 대한 다형성을 구현한다는 공통점이 있지만 구현된 객체의 메모리 할당량은 다를 수 있습니다.

변수가 특정 객체를 저장한다는 것은 객체의 메모리 주소를 갖는 포인터를 저장하고 있다는 의미입니다.

만약 Opaque Type 으로 선언된 변수가 저장해야 할 변수의 메모리 타입이 너무 크다면 미리 준비한 메모리에는 저장할 수 없으니 별도의 포인터를 참조하는 수밖에 없습니다.

두 키워드의 관계는 다음과 같이 나타낼 수 있습니다.

`변수에 any 키워드로 객체 저장 -> some 키워드로 선언된 변수로 옮기기 시도 -> 컴파일러가 any 로 선언된 변수에서 구체 타입을 꺼내 some 키워드 변수로 옮기기`

## some, any 를 이용한 추상화가 필요한 이유

WWDC 의 예시는 동물 객체의 eat 메소드가 비슷한 boiler plate 코드로 구현된다는 점을 들었습니다.

```swift
// No Abstraction
struct Farm {
  func feed(_ animal: Cow) {
    let alfafa = Hay.grow()
    let hay = alfalfa.harvest()
    animal.eat(hay)
  }
  
  func feed(_ animal: Horse) {
    let root = Carrot.grow()
    let carrot = root.harvest()
    animal.eat(carrot)
  }
  
  func feed(_ animal: Chicken) {
    let wheat = Grain.grow()
    let grain = wheat.harvest()
    animal.eat(grain)
  }
}

// Admit Abstraction
struct Farm {
  func feed(_ animal: some Animal) {
    let crop = type(of: animal).Feed.grow()
    let produce = crop.harvest()
    animal.eat(produce)
  }
}
```

## Summary

some, any 키워드는 추상화된 코드를 작성할 때 유용하게 사용할 수 있는 키워드입니다.

some 은 Underlying 타입을 표현할 때 사용하고, any 는 런타임에 Opaque 타입을 구현한 Underlying 타입들을 모두 표현할 때 사용합니다.

some 은 특정 프로토콜을 구현한 객체를 파라미터, 리턴값, 프로퍼티로 선언하는 경우에 주로 사용하게 됩니다.

any 키워드는 런타임에 따라 특정 Underlying 타입으로 바뀔 수 있는 프로토콜 타입의 Element 를 저장하는 배열을 선언할 때 주로 사용할 수 있습니다. 

some, any 키워드를 구분해서 사용해야 하는 이유는 코드를 실행할 때 메모리 사용량을 미리 알 수 있어야 하기 때문입니다. Opaque 타입으로 구현된 객체들의 메모리 사용량이 각기 다를 수 있기 때문입니다. 

Reference:
* [WWDC 2022 - Embrace Swift generics](https://developer.apple.com/videos/play/wwdc2022/110352/)
* [Swift Docs - Opaque Types](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)