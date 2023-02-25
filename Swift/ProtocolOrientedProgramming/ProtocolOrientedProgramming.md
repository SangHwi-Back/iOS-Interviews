# Protocol-Oriented-Programming (POP) in Swift

OOP 에서 문제가 된 사항 중 하나는 Super Class 를 상속한 객체를 다중 스레드 환경에서 사용하게 될 경우 예기치 않은 원본 데이터 수정이 발생할 수 있다는 점이었습니다.

Swift 의 대부분은 구조체입니다. 이는 참조 타입은 Class 타입을 사용하지 않음으로써 앞에서 얘기한 `예기치 못한 수정 을 방지` 하고 `Reference Cycle 을 방지` 방지하기 위함으로 생각됩니다. 

그렇지만, 공통 기능을 모듈화 한다는 점에 있어서 OOP 의 상속과 관련된 기능을 버리기에는 너무 아깝습니다. 이럴 때 사용할 수 있는 것이 protocol, extension 입니다.

## Protocol

protocol 은 특정 메소드 및 프로퍼티에 대한 blueprint 를 제공하는 역할을 합니다. 만약 특정 class, struct, enum 이 protocol 을 채택하여 구현하게 된다면 구현 객체들은 protocol 로서 표현될 수 있습니다. 여기서 protocol 을 채택하여 구현한 타입을 conforming type 이라고 부릅니다.

protocol 만을 사용할 때의 단점은 공통되는 기능이라도 blueprint 만을 제공할 수 있다는 점입니다. 즉, 공통 기능을 반복적으로 구현해야 한다는 단점이 있습니다.

이럴 때 protocol extension 을 사용할 수 있습니다. 특정 protocol 을 타입처럼 extension 하면 blueprint 에서 사용할 일부 기능을 미리 구현해 놓을 수 있습니다.

protocol 에서 제공할 수 있는 필수 구현 조건(blueprint) 에는 property, method, subscript 가 있습니다.

## Protocol Generics

protocol 에서 사용할 수 있는 키워드 중 `associatedtype` 은 conforming 타입에서 사용할 Generic 타입을 선언할 수 있습니다.

## Example

```swift
protocol HTTPRequestProtocol {
    associatedtype DecodedType
}

extension HTTPRequestProtocol where DecodedType: Decodable {
    /// request 함수를 공통적으로 사용할 수 있도록 미리 구현해 놓습니다.
    /// 
    /// DecodedType 이 반드시 Decodable 을 채택하고 있는 타입이어야 합니다.
    mutating func request(to url: URL, _ completionHandler: @escaping (DecodedType?) -> Void) {
        URLSession.shared.dataTask(with: URLRequest(url: url)) { data, response, error in
            guard let data = data else {
                completionHandler(nil)
                return
            }
            
            do {
                completionHandler(try JSONDecoder().decode(DecodedType.self, from: data))
            } catch {
                completionHandler(nil)
                print(error)
            }
        }
    }
}

/// DecodedType == AnonymousJSON
class HTTPModel<AnonymousJSON>: HTTPRequestProtocol { }

let model = HTTPModel<AnonymousJSON>()
var item: AnonymousJSON?
model.request(to: URL(string: "https://......")!) { result in
    item = result
}     
```

References
* [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/)