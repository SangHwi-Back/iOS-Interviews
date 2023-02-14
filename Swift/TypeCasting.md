> 소스코드는 직접 작성하였습니다. [Swift Docs - TypeCasting](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html) 를 참고하셔도 무방합니다.

# Type Casting

타입 캐스팅이란 객체의 타입을 체크하거나 객체를 클래스 계층안에서 다른 슈퍼/서브 클래스로 다루는 방법을 의미합니다.

타입 캐스팅을 다룰 때 사용하는 연산자는 두 개가 있는데 `is`, `as` 가 그것입니다. 이  연산자들로 값의 타입을 체크하거나 다른 타입으로 캐스팅하는 것이 가능합니다.

## Defining a Class Hierarchy for Type Casting(Subclassing)

타입 캐스팅은 클래스가 서브클래싱을 통해 형성하는 클래스 계층구조 내에서 객체를 사용할 수 있게 합니다.

이를 이해하기 위해 먼저 클래스 계층구조를 형성하는 방법에 대해 아래와 같이 소개합니다.

```swift
class Animal {
    var name: String 
    init(name: String) {
        self.name = name
    }
}

class Tiger: Animal {
    var isPatterned: Bool
    init(name: String, isPatterned: Bool) {
        self.isPatterned = isPatterned
        super.init(name: name)
    }
}

class Horse: Animal {
    var bodyColor: UIColor
    init(name: String, bodyColor: UIColor) {
        self.bodyColor = bodyColor
        super.init(name: name)
    }
}

let animals = [
    Animal(name: "Human")
    , Tiger(name: "Derek", isPatterned: true)
    , Tiger(name: "Jenny", isPatterned: false)
    , Horse(name: "Willson", bodyColor: .white)
    , Horse(name: "Sarah", bodyColor: .black)
]
```

* Animal 클래스는 name 프로퍼티를 갖습니다.
* Tiger 클래스는 isPatterned 프로퍼티를 갖고, 슈퍼 클래스인 Animal 의 생성자를 호출하기 위해 name 파라미터를 전달합니다.
* Horse 클래스는 bodyColor 프로퍼티를 갖고, 슈퍼 클래스인 Animal 의 생성자를 호출하기 위해 name 파라미터를 전달합니다.
* animals 배열의 타입은 `[Animal]` 로 유추 됩니다.

## Checking Type(Operators)

객체가 특정 서브클래스 타입인지 확인하고 싶다면 `is` 를 사용합니다.

`is` 연산자의 결과 값은 Bool 입니다.

```swift
for animal in animals {
    if animal is Tiger {
        print("Look out! There is a Tiger!")
    } else if animal is Horse {
        print("Wow! What a nice Creature!")
    }
}
```

## Downcasting(Operators)

객체의 특정 타입은 사실 서브클래스일 가능성이 있습니다. 이러한 경우 Downcasting 을 통해 타입을 확인해볼 수 있는데, `as?` / `as!` 두 가지로 확인 가능합니다.

* as? : 해당 연산자로 타입 캐스팅을 시도하면 의도한 타입의 옵셔널 값을 반환합니다. 즉, 실패 시 nil 을 반환하는 것입니다.
* as! : 해당 연산자로 타입 캐스팅을 시도하면 의도한 타입의 값을 반환합니다. 이 경우 타입 캐스팅에 실패하면 런타임 에러가 발생합니다.
* 주의 : Downcasting 작업이 객체의 값을 변경하지는 않습니다.

> C / C++ 언어에는 Superclass 로 타입캐스팅 하는 경우 Upcasting, Subclass 로 타입캐스팅 하는 경우 Downcasting 으로 나누기도 하지만, Swift 언어의 공식문서에는 Upcasting 에 대한 용어를 발견할 수 없었습니다.

주로 Downcasting 은 Optional Binding 을 이용합니다. 사용하는 방법은 if-clause 에 if let 을 아래와 같이 이용하는 것입니다.

```swift
for animal in animals {
    if let tiger = animal as? Tiger {
        print("This tiger is named \(tiger.name). \(tiger.isPatterned ? "Look at this beautiful pattern!" : "This body is like marble!")")
    } else if let horse = animal as? Horse {
        print("\(horse.name)! Come here! Not you! You are not \(horse.bodyColor)!")
    }
}
```

if let 을 사용한 if-clause 내부에(if-clause 내부의 scope) Downcasting 된 변수는 옵셔널이 아닙니다.

> Swift 는 코드를 작성할 때 실제 문장처럼 보이도록 작성하도록 권장합니다. 'if let tiger = animal as? Tiger' 은 여러가지 방식으로 작성될 수 있지만, '만약 tiger 가 animal 을 Downcasting 한 Tiger 라면...' 처럼 읽히도록 코드를 작성하는 것을 권장합니다.

## Type Casting for Any and AnyObject

Swift 는 두 개의 정의되지 않은 타입을 갖습니다.

* Any : 객체가 모든 타입으로 표현될 수 있을 경우 부여하는 타입입니다. 함수 타입에도 사용 가능합니다.
* AnyObject : 객체가 클래스이면서 모든 타입으로 표현될 수 있을 경우 부여하는 타입입니다.

```swift
var uiElements: [AnyObject] = []

uiElements.append(UIView())
uiElements.append(UIButton())
uiElements.append(UISwitch())
uiElements.append(UIStackView())
uiElements.append(UITableView())
uiElements.append(false) // 컴파일 에러 발생 : Argument type 'Bool' expected to be an instance of a class or class-constrained type

var things: [Any] = []

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })
```

Obj-c 기반의 API 등이 아니면 대부분 Any 를 사용하고 있습니다.

이렇게 여러가지 상황에서 타입캐스팅을 해야 할 경우에는 is, as 를 switch 문 안에서 사용할 수 있습니다.

```swift
for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}
```

참고로 Any 는 옵셔널 타입도 표현 가능합니다. 그러므로 아래와 같이 옵셔널 Int 를 Any 배열에 추가하려고 할 때 다음과 같이 처리할 수 있습니다.

```swift
let optionalNumber: Int? = 3
things.append(optionalNumber)        // Warning
things.append(optionalNumber as Any) // No warning
```

References:
* [Swift Docs - TypeCasting](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)
