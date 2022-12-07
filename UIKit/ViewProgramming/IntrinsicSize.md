본질적인 컨텐츠 크기이다.
실제 뷰의 크기가 아닌 뷰가 가지고 있는 컨텐츠의 크기이다. 예를 들어 UILabel의 경우 text가 보여지는 크기를 말한다.
하지만, invalidateIntrinsicContentSize()를 호출하면 intrinsicContentSize:CGSize 프로퍼티를 수정할 수 있다.
내부적으로 intrinsicContentSize는 updateConstraints를 호출한 직후이다.
