@main, UIApplicationMain 함수에 대해 설명하시오.

이 함수는 두 가지 역할이 있다.
1. 애플리케이션 객체를 최초의 객체로서 초기화 한 뒤 주어진 AppDelegate 클래스에 델리게이트 한다.
2. Main Event Loop와 App 자체의 Run Loop를 설장하고 실행한다.

Info.plist 파일이 main nib 파일 형태로 준비되어 있다면, NSMainNibFile을 키로 삼아 준비된 파일을 불러 오도록 할 수 있다.
