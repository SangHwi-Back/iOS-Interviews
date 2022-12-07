상태 변화에 따라 다른 동작을 처리하기 위한 앱델리게이트 메서드들을 설명하시오.

앱의 상태변화에 따른 앱의 동작을 정의한다. 여기서 말하는 상태변화에는 Foreground(Active+Inactive), Background, Unattached, Suspended가 있다.
앱을 iOS의 통제 하에 두기 위해 프로젝트에 기본으로 제공되는 AppDelegate.swift 파일에는 @main 어노테이션을 실행시키는 코드가 기본으로 제공된다.
application(_:UIApplication, didFinishLaunchingWithOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool  App실행 직후 사용자의 화면에 보여지기 직전 실행되는 메소드이다.
application(_:UIApplication, willFinishLaunchingWithOptions: [UIApplicationLaunchOptionsKey: Any]? = nil) -> Bool  App이 최초 실행될 때 호출되는 메소드이다.
applicationWillResignActive(_application:UIApplication) InActive 상태로 전환되기 직전 호출된다.
applicationDidEnterBackground(_application:UIApplication) Background 상태로 전환되기 직전 호출된다.
applicationWillEnterForeground(_application:UIApplication) Active 상태가 되기 직전, 화면에 보여지기 직전 호출됨.
applicationDidBecomeActive(_application:UIApplication) Active 상태로 전환된 직후 호출
applicationWillTerminate(_application:UIApplication) App이 종료되기 직전에 호출

---

![](../_Images/AppDelegate_SceneDelegate_Image1.png)

SceneDelegate 에 대해 설명하시오.

iOS 13 이상부터 앱의 상태관리를 담당하는 객체가 따로 만들어졌는데 이것이 SceneDelegate 이다. iOS 13 이전에는 AppDelegate 의 window 변수(첫번째 생성되는 부모 view)가 SceneDelegate 의 scene 으로 대체된다. 이를 통해 SceneDelegate 를 사용하는 이유 중 하나는 iOS 및 iPadOS 의 다중 창 앱 빌드가 가능하다.
SceneDelegate의 등장으로 인해 Scene Session이라는 개념이 생겨나면서 iPad 에서는 여러 개의 앱을 동시의 띄울 수 있는 것이 가능해졌다. AppDelegate의 프로세스 하나에 여러 개의 UI(Scene Session)을 띄우는 것이다.
scene(_:willConnectTo:options:)  scene이 앱에 추가될 때 호출.
sceneDidDisconnect(_:)  scene의 연결이 해제될 때 호출. 재 연결도 가능하다.
sceneDidBecomeActive(_:)  app Switcher에서 선택되거나, scene간의 상호작용에 의해 호출.
sceneWillResignActive(_:)  scene과 사용자의 상호자가용이 중지될 때 호출.
sceneWillEnterForeground(_:)  scene이 foreground에 진입할 때 호출.
sceneDidEnterBackground(_:)  scene이 백그라운드로 진입할 때 호출.
