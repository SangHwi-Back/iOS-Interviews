## KVC

객체의 값을 가져오기 위해 Key, KeyPath를 이용해서 간접적으로 데이터를 접근하여 불러오거나 수정하는 방법.

KeyPath 문법 = \BaseType.PropertyName
KeyPath의 종류 = KeyPath(Read-Only), WritableKeyPath(Struct 등 Value type 인스턴스에 사용. Read&Write), ReferenceWritableKeyPath(Class 등 Reference type 인스턴스에 사용. Read&Write).

Key-Value Coding. 객체의 값을 직접 수정하는 것 보단, Key 또는 KeyPath 를 이용해서 간접적으로 데이터를 가져오거나 수정하는 방법. KeyPath는 \BaseType.PropertyName 으로 만든다.
KeyPath 타입으로 프로퍼티 접근 시 읽기 전용으로 값 접근이 가능하다.
WritableKeyPath 으로 프로퍼티 접근 시 프로퍼티 읽기/쓰기가 가능하다.
값 타입(struct)에서는 WritableKeyPath 를 그대로 사용하고, 참조 타입(class)의 경우 ReferenceWritableKeyPath 라는 타입의 KeyPath 를 사용한다.

## KVO

인스턴스의 속성에 대한 변경 사항을 다른 객체에 알리기 위한 Cocoa 프로그래밍 패턴. 논리적으로 분리(예: 모델-뷰)된 앱 부분 간의 변경사항을 전달하는 데 유용하다. NSObject를 상속하는 클래스만 Observe 가능(Objective-C 런타임 사용 필요).

객체의 프로퍼티와 변경사항을 다른 객체에 알리고 싶은 경우 사용하는 코코아 프로그래밍 패턴.
Objective-C 런타임을 사용해야 하기 떄문에 @objc 을 사용하고 dynamic modifier 를 붙여야 한다.
참고로 Objective-C 런타임을 사용하는 이유는 함수 등의 구현을 앱 런타임에 결정하기 위해 사용한다.
@objc dynamic 을 붙인 프로퍼티에 대한 Keypath 와 옵션을 넣어서 .observe 메소드를 호출하면 된다. KVO 로 관찰되어야 하는 프로퍼티가 포함된 타입은 NSObject 를 상속해야 한다.

사용방법

Observe가 필요한 프로퍼티에 @objc dynamic 을 붙인다.

```swift
class Dog: NSObject {
    @objc dynamic var name: String
    init(name: String) {
        self.name = name
    }
}
```

Observer를 설정한다. 관찰하려는 객체를 참조하는 변수의 observe 함수를 이용한다. 파라미터로는 KeyPath, Options, escaping closure가 필요하다.

```swift
var dog = Dog(name: "bingo")
dog.observe(\.name, options: [.old, .new]) { (object, change) in
    print("old is (change.oldValue), new is \(change.newValue)")
}
dog.name = "mango"
```
