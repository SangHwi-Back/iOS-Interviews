App의 Not running, Inactive, Active, Background, Suspended에 대해 설명하시오.

App 의 InActive/Active 는 묶어서 Foreground, Background/NotRunning 은 묶어서 Background 라고 부른다.
App 은 Suspended 에서 시작해서 InActive, Active 순으로 실행된다.
만약 앱이 Foreground에서 벗어나려면 우선 InActive 상태가 된 후 Background 상태로 접어들게 된다. 이후에는 iOS 시스템에 의해 Not-Running 혹은 Suspended 된다.

Reference:
* [Apple Documentation - Managing your app's life cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
* [Apple Documentation - Preparing your UI to run in the foreground](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground)
* [Apple Documentation - Preparing your UI to run in the background](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)