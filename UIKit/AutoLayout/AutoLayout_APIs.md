1. NSLayoutConstraint (iOS 6)

NSLayoutConstraint 클래스를 사용하여 2 개의 UI 객체에 대해 레이아웃을 충족시킬 수 있는 관계식을 성립시키는 방식으로 UI를 작성할 수 있다. 많은 파라미터를 지정해야 하기 때문에 코드가 길어질 수 있다.

2. NSLayoutAnchor (iOS 9)

NSLayoutConstraint는 간접적으로 2개의 뷰 레이아웃 관계식을 정의하는 것이었는데, NSLayoutAnchor는 원하는 객체에 직접적으로 뷰 레이아웃 관계식을 정의하기 위해 만들어진 API이다. NSLayoutConstraint를 만드는 다른 방법이다.

UIView는 관계식을 정의할 수 있는 요소마다 Anchor를 갖는다. Anchor 프로퍼티는 constraint 함수를 이용해서 NSLayoutConstraint를 생성하고 isActive 프로퍼티를 true로 설정하면 관계식이 성립된다. 참고로 margin은 UIView의 layoutMarginsGuide 프로퍼티를 사용해야 한다.

NSLayoutConstraint 에 비해 필요한 부분에만 업데이트를 한다는 점에서 코드가 훨씬 간소화 된다.

3. Visual Format Language

각종 ASCII를 이용하여 제약 조건을 정의한다.

