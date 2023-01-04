# some, any 키워드에 대해 아시는대로 말씀해주세요.

some 키워드와 any 키워드는 Swift 추상화를 구현할 때 유용한 키워드입니다. 여러 개의 반복적인 boiler-plate 코드를 만드는 것은 관리적인 측면이나 작성하는 측면에서 모두 비용이 많이 드는 일이기 때문에 추상화는 아주 유용합니다.

추상화된 타입을 통해 하나의 코드가 여러 방식으로 작동하는 것을 '다형성' 이라고 합니다. 추상화를 통한 다형성을 가진 코드를 작성하는 방법은 세 가지가 있습니다.

* Overriding Morphism - 오버라이딩을 통한 다형성
* Sub type Morphism - 하위 타입을 이용한 다형성
* Parametric type Morphism - 매개변수를 이용한 다형성

이 중 some, any 키워드는 `Parametric type Morphism` 을 구현하는 데 아주 유용합니다.

some, any 키워드가 구현하는 Opaque 타입에 대해 알아보는 것부터 시작하도록 하겠습니다.

## Opaque, Underlying 타입

구체 타입의 place holder 를 나타내는 타입을 Opaque(불투명) 타입이라고 말합니다. 대체되는 특정 구체 타입을 Underlying(기본) 타입이라고 말합니다.

즉, Opaque Type -> Underlying Type 이라고도 말할 수 있겠습니다.

Opaque 타입을 사용할 경우 값은 Underlying 타입으로 한정되어 Underlying 타입을 얻는 것을 보장받을 수 있습니다.

```swift
let chicken: some Animal = Chicken() // Underlying 타입을 반영하여 타입 범위를 Underlying 으로 한정함.
chicken = Horse() // Error!! 한정된 Underlying 타입이 아님.
```

Opaque 타입 추론은 호출하는 부분, 반환하는 부분에서 가능합니다.

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

여기서 주의해야 할 사항은 Underlying 타입으로 `한정` 된다는 것입니다. 다음과 같은 상황에 대해서는 컴파일 에러가 발생합니다.

```swift
func checkAnimal(at farm: Farm) -> some Animal {
  if farm.location == Location.north {
    return Chicken()
  } else {
    return Cow()
  }
}
```

> 참고 : SwiftUI 에서 `var body: some View` 의 경우 여러가지 뷰를 리턴할 수 있는데, 잘 보면 return 키워드가 없습니다. @ViewBuilder 가 제공하는 DSL 기능의 하나로, 각 조건에 따른 Underlying 타입을 사용하도록 빌드할 수 있습니다.

이런 타입은 모듈이나 코드를 작성함에 있어 유리한데, 호출하는 쪽에서 기본 타입(Underlying Type) 을 private 하게 소유할 수 있기 때문이다.

## some 키워드

some 키워드는 적합성의 기준으로 판단되는 프로토콜을 구현한 타입인 Opaque 타입을 선언할 때 사용됩니다.

Swift 5.7 부터 some 키워드는 property, parameter, return 타입을 표현할 때 사용할 수 있습니다.

some 키워드 내에서 사용되는 Underlying 타입은 하나로 고정됩니다.

```swift
[some ProtocolName] // X
func checkSum(_ numbers: [some ProtocolName]) -> some CustomResultType { ... }// X

[any ProtocolName]
func checkSum(_ numbers: [any ProtocolName]) -> some CustomResultType { ... }
```

## any 키워드

any 키워드는 some 키워드에 의해 고정된 Underlying 타입의 한계를 극복합니다. 예를 들어 Animal 프로토콜을 구현한 객체를 저장하는 배열을 선언한다고 해보겠습니다.

`[some Animal]`

이 코드는 컴파일 에러를 일으킵니다. `[some Animal]` 은 `Cow`, `Horse`, `Cat` 등 모든 객체를 포함시키고 싶지만 위에서 말한대로 하나의 Underlying 타입으로 고정됩니다.

그러므로 아래와 같이 표현해야 합니다.

`[any Animal]`

즉 any 키워드는 특정 Opaque 타입이 모든 종류의 Underlying 타입을 표현할 수 있다고 선언하는 것과 같습니다. 그리고 Underlying 타입이 런타임마다 달라질 수 있다는 것도 선언하는 것과 같습니다.

Reference:
* [WWDC 2022 - Embrace Swift generics](https://developer.apple.com/videos/play/wwdc2022/110352/)
* [Swift Docs - Opaque Types](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)