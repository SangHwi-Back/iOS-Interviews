## KVC

객체의 값을 가져오기 위해 Key, KeyPath 를 이용해서 간접적으로 데이터를 접근하여 불러오거나 수정하는 방법을 일컫습니다.

* 문법 = `\BaseType.PropertyName`
* 종류
  1. KeyPath(Read-Only)
  - KeyPath 타입으로 프로퍼티 접근 시 가져온 값은 읽기 전용입니다.
  3. WritableKeyPath(Struct 등 Value type 인스턴스에 사용. Read&Write)
  - WritableKeyPath 으로 프로퍼티 접근 시 프로퍼티 읽기/쓰기가 가능합니다. 
  5. ReferenceWritableKeyPath(Class 등 Reference type 인스턴스에 사용. Read&Write)
  - 접근하는 값이 Class 등 참조 타입일 경우 사용합니다. 읽기/쓰기 모두 가능합니다.

## KVO

인스턴스의 속성 변경을 다른 객체에 알리기 위한 Cocoa 프레임워크에서 제공하는 프로그래밍 패턴입니다.

논리적으로 분리(예: 모델-뷰)된 객체 간에 변경사항을 공유하는 데 유용합니다.<br/>
NSObject 를 상속하는 클래스에서만 Observe 가능합니다(Objective-C 런타임 사용 필요).

@objc dynamic 을 붙인 프로퍼티에 대한 Keypath 와 옵션을 넣어서 .observe 메소드를 호출하면 된다. KVO 로 관찰되어야 하는 프로퍼티가 포함된 타입은 NSObject 를 상속해야 한다.

사용방법은 다음과 같습니다.

1. 게시자(이벤트 발생자)는 Obj-C 런타임 사용을 위해 @objc 과 dynamic 수정자를 이용해야 합니다.
2. 관찰자(Observe)는 KeyPath(KVO) 를 이용하여 관찰할 프로퍼티를 지정합니다.

```swift
// 게시자
class Dog: NSObject {
    @objc dynamic var name: String
    init(name: String) {
        self.name = name
    }
}

var dog = Dog(name: "bingo")

// 관찰 시작
dog.observe(\.name, options: [.old, .new]) { (object, change) in
    print("old is (change.oldValue), new is \(change.newValue)")
}
dog.name = "mango"
```

Observe 를 등록할 때 옵션을 통해 여러가지 방식으로 인스턴스의 변경을 관찰할 수 있습니다.