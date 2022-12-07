Safe areas는 시각적으로 보장된 곳에 뷰들을 위치시키기 위해 필요하다. Navigation Bar 등에 의해 컨텐츠가 가려지는 현상을 방지하면서 constraint를 부여할 수 있다. AutoLayout을 사용하지 않는 경우는 safeAreaInsets 프로퍼티를 사용하고 AutoLayout을 사용하는 경우는 safeAreaLayoutGuide 프로퍼티를 사용한다.

만약 UIViewController 안에 child UIViewController가 포함된 경우 child VC의 Safe area를 업데이트하여 컨테이너가 다루는 영역을 제외해줘야 한다. 예를 들어 UIKit은 이미 자신의 child UIViewController들의 Safe area를 조정해 놓았다. 공식문서에서는 이를 extending the safe area 라고 언급하였다.

아래는 viewDidAppear(_:)에서 extend the safe area 하는 코드 예시이다.

```swift
override func viewDidAppear(_ animated: Bool) {
    var newSafeArea = UIEdgeInsets()

    if let sideViewWidth = sideView?.bounds.size.width {
        newSafeArea.right += sideViewWidth
    }

    if let bottomViewHeight = bottomView?.bounds.size.height {
        newSafeArea.bottom += bottomViewHeight
    }

    let child = self.children[0]
    child.additionalSafeAreaInsets = newSafeArea
}
```

[https://developer.apple.com/documentation/uikit/uiview/positioning_content_relative_to_the_safe_area](https://developer.apple.com/documentation/uikit/uiview/positioning_content_relative_to_the_safe_area)
