## attribute - Declaration Attributes

available 은 Swift 버전이나 플랫폼, OS 버전 등에 따라 선언을 제어하기 위해 사용됩니다.

available 은 2개 이상의 args 를 포함하고 있습니다. 첫번째는 플랫폼이나 swift 버전을 명시하고, 두번째부터는 제한할 특정 정보를 표시하기 시작합니다. 

다음은 available Attribute 에 넣을 수 있는 args 목록입니다.

1 번째 Comma 전

* platform = 다음 중 하나를 선택하여 넣을 수 있습니다. 아래 swift 버전을 제외하고는 * 와일드카드를 사용하여 모든 플랫폼에서 가능하도록 할 수도 있습니다.
    * iOS
    * iOSApplicationExtension
    * macOS
    * macOSApplicationExtension
    * macCatalyst
    * macCatalystApplicationExtension
    * watchOS
    * watchOSApplicationExtension
    * tvOS
    * tvOSApplicationExtension
    * swift

1 번째 Comma 후

* availability = 가능, 불가능 여부를 정의합니다.
  * `available`, `unavailable`
* introduced = 특정 플랫폼이나 언어에서 도입된 첫 번째 버전을 나타냅니다.
  * `introduced: "version number"`
* deprecated = 특정 플랫폼이나 언어에서 해당 선언은 사용 가능하나, 추천되지는 않는 버전을 나타냅니다. 만약 버전을 나타내지 않는다면 현재 버전을 참조하게 됩니다.
  * `deprecated: "version number"`
* obsoleted = 특정 플랫폼이나 언어에서 더 이상 사용할 수 없음을 나타냅니다.
  * `obsoleted: "version number"`
* message = 컴파일러가 경고나 에러를 표시할 때 표시하는 메시지를 정의합니다. 경고나 에러는 deprecated, obsoleted 등을 어겼을 경우 나타납니다.
  * `obsoleted: "version number"`
* renamed = 해당 선언의 이름이 바뀌었을 경우 어떤 이름으로 바뀌었는지 표현할 때 사용합니다. unavailable 속성과 함께 사용됩니다.
  * `@available(*, unavailable, renamed: "NameRenamed")`

available Attribute 는 동시에 여러 개 사용할 수 있습니다. 또한 introduced 정보를 포함할 때는 줄여서 사용할 수도 있습니다.

```swift
// iOS 14, swift 4.0 버전부터 사용 가능.
@available(iOS 14.0, *)
@available(swift 4.0)
```

```swift
// iOS 10, macOS 10.12 버전 이후로 사용 가능.
@available(iOS 10.0, macOS 10.12, *)
```

```swift
// iOS 14, swift 4.0 버전부터 사용 가능. swift version, platform info 를 같이 사용할 때는 한 줄로 사용할 수 없습니다.
@available(swift 4.0)
@available(iOS 14.0, *)
```

## 참고

`#available` 은 다릅니다. if 등의 조건문에서 조건을 나타낼 때 사용됩니다.

```swift
if #available(iOS 13.0, *) {
    return 1
} else {
    return 0
}
```

Reference:
* [Swift - Language Reference, Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)