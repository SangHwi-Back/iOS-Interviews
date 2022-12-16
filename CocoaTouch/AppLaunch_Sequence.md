App 실행과정

1.	main 함수 실행 후 @UIApplicationMain 함수를 호출 함.
2.	@UIApplicationMain 함수에 의해 UIApplication 객체를 생성함. UIApplication 객체는 앱의 본체에 해당한다.
3.	nib 파일, info.plist 등 파일을 읽어들여 앱 실행에 필요한 데이터를 로드한다.
4.	AppDelegate, SceneDelegate 를 만들고 UIApplication 객체와 연결된 런 루프를 만드는 등 실행에 필요한 준비를 한다.
5.	AppDelegate 의 application(_application:UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool 을 실행한다.

Reference: 
* [Apple Documentation - Responding to the launch of your app](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app)