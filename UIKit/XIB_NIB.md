# xib, nib 파일에 대해 설명해주세요.

## 정의

두 파일의 이름은 다음의 약자입니다.

* NIB = Next Interface Builder
* XIB = XML Interface Builder

둘 다 화면 UI 를 기록하는 파일이라는 공통점이 있지만 파일 구조에 차이점이 있습니다. nib 는 바이너리 형태를 띄지만, xib 는 XML 문법을 따르기 때문에 SCM(소스 제어 관리) 시스템으로 관리하기가 쉽습니다.

기본적으로 역할은 UI Elements 를 만드는 것이기 때문에 하나의 xib, nib 파일에는 여러 개의 뷰를 만들 수 있습니다.

빌드를 진행하게 되면 xib 는 nib 로 컴파일 됩니다.

## 뷰 라이프사이클과의 관계

빌드를 통해 nib 파일로 컴파일 된 경우 앱에서는 UI Elements 를 nib 파일에서 Unarchive 처리하여 불러오게 됩니다.

이 경우 아래와 같은 라이프사이클 함수에서 그 변화를 관찰할 수 있습니다.

* 뷰 객체를 모두 Unarchive 처리한 직후
```swift
override class func awakeFromNib() {
  super.awakeFromNib()
}
```
* 뷰 객체를 Unarchive 처리하려고 시도할 때
```swift
required init?(coder: NSCoder) {
  super.init(coder: coder)
}
```
* 뷰 객체를 직접 생성하려고 할 때
```swift
override init(frame: CGRect) {
  super.init(frame: frame)
}
```

개인의 판단에 따라 각각의 상황에서 처리해야 할 작업이 있을 경우에는 super 클래스에서 해당 라이프사이클 메소드를 실행한 후 작업하는 것이 안전하다.

## 언제 사용해야 하는가?

xib, nib 파일들은 별도의 프로그램이었다가 Xcode 에 Storyboard 를 열 때 사용되는 "인터페이스 빌더" 에서 사용됩니다.

Xcode 에서는 View 타입의 파일을 만들게 되면 xib 파일이 생성됩니다.

iOS 는 MVC 패턴을 기본적인 디자인 패턴으로 사용하고 있지만, ViewController 를 뷰라고 부를 정도로 뷰의 존재감이 많이 없습니다. 이 때 대안이 될 수 있는 것이 XIB, NIB 를 사용하는 Storyboard 를 뷰로 가정하고 개발하는 것입니다.

뷰 자체를 따로 작업할 수 있기 때문에 재사용성이 높고, Preview 가 가능하며 동적 로딩이 가능하다는 점은 장점으로 인식됩니다.

하지만, UI 의 구성이나 레이아웃이 동적으로 변하거나 뷰 컨트롤러 사이의 연결(Transition) 구현은 할 수 없으며 인터페이스 빌더 환경에서 바꿀 수 없는 속성이 있어서 코드를 통해 구현해야 하는 경우가 있습니다.

Reference:
* [iOS ) nib과 xib의 차이](https://zeddios.tistory.com/298)
