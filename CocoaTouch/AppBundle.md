> Summary 만 봐도 됩니다. 깊이 공부하기 위해 애플 공식문서 아카이브를 파고 들었습니다. 대부분의 내용을 담으려 하였지만 개인 판단에 따라 가독성을 고려하여 누락시킨 부분도 있습니다.

## App Bundle의 구조와 역할에 대해 설명하시오.

Bundle 구조의 정의는 플랫폼이나 Bundle 타입에 따라 개념과 구조가 상이하다. 이 문서에서는 iOS 에서 채택하는 Application 의 Bundle 만을 다룬다.

### Application Bundles - 애플리케이션 번들

Application Bundles 구조는 개발자에 의해 만들어지는 가장 기본적인 타입의 Bundle 구조이다. Bundle 은 애플리케이션 동작에 필요한 모든 종류의 요소를 담고 있다.

#### What File Go Into an Application Bundle? - 애플리케이션 번들의 파일 종류

아래 표는 Application Bundle 안에 들어간 파일 타입들을 나타낸다. 파일들의 정확한 위치는 플랫폼마다 다양하고 일부는 누락되어 있을 수도 있다.

| File                | Description                                                                                                                                                                                            |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Info.plist          | **(Required)** App 설정정보를 구조적으로 담은 파일.<br/>시스템은 앱에 필요한 정보가 있을 경우 이 파일을 참고한다.                                                                                                                            |
| Executable          | **(Required)** 앱의 실행 파일. <br/>앱의 main entry point, application target 의 고정된 링크를 포함한다.                                                                                                                  |
| Resource files      | 위의 Executable 을 제외한 데이터 파일들을 통칭. 다국어 적용이 가능하다.<br/>Image, Icon, Music, Nib, strings, configuration 등 데이터 파일들의 모음이다.<br/>Resource 의 구조 및 요소는 플랫폼에 따라 상이하다.                                              |
| Other support files | mac 의 경우 private framework, plug-ins, document templates, custom data resources 들을 앱 번들 구조에 포함할 수 있다.<br/>iOS 의 경우 custom data resources 는 번들 구조에 포함시킬 수 있지만, custom framework 나 plug-ins 는 포함시킬 수 없다. |

#### Anatomy of an iOS Application Bundle - iOS 애플리케이션 번들 세부구조

iOS 애플리케이션 번들 구조는 당연하게도 모바일 애플리케이션에 적합하다. 비교적 평평한 구조로 외부 디렉토리를 최소한으로 하여 디스크 용량과 파일 접속을 최소화한다.

**The iOS Application Bundle Structure - iOS 애플리케이션 번들 구조**

애플리케이션의 Executable, Resources(Icon/Images/Localized File)이 최상위 번들 디렉토리에 위치한다.

en.lproj, fr.lproj 파일처럼 다국어가 적용된 파일들은 서브 디렉토리에 위치하게 된다.

> MyApp.app
> - MyApp **(Executable)**
> - MyAppIcon.png **(Icon)**
> - MySearchIcon.png **(Icon)**
> - Info.plist **(Info.plist)**
> - Default.png **(Resource files)**
> - MainWindow.nib **(Resource files)**
> - Settings.bundle **(Other support files)** - Setting 앱 플러그인에 필요한 특별 Bundle.
> - MySettingsIcon.png **(Icon)**
> - iTunesArtwork **(Resource files)**
> - en.lproj **(Resource files)**
>   - MyImage.png **(Resource files)**
> - fr.lproj **(Resource files)**
>   - MyImage.png **(Resource files)**

다국어 버전을 제공하지 않더라도 한 개의 디렉토리(기본 언어 설정)는 존재해야 한다.

**The Information Property List File - Info.plist 파일**

Info.plist 파일은 애플리케이션의 설정 정보를 가지고 있다. Xcode 에서 프로젝트를 생성하면 기본 형태의 Info.plist 가 기본 제공되지만 아래의 요소들은 직접 정의해주어야 한다.

| Key                                                             | Value                                                                                                                                                 |
|-----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| CFBundleDisplayName<br/>(Bundle display name)                   | Application Icon 밑에 위치하는 이름. 다국어 적용 가능.                                                                                                               |
| CFBundleIdentifier<br/>(Bundle identifier)                      | 시스템이 앱을 인식하는 데 필요한 Identifier. UTI 구조를 따른다.<br/>DNS 구조의 반대의 형태를 따라야 한다. <br/>(예)DNS = Ajax.com , App Name = Hello, Bundle Identifier = com.Ajax.Hello |
| CFBundleVersion<br/>(Bundle version)                            | bundle 의 build version 을 표시. 증가하는 문자열이다. 다국어 적용되지 않는다.                                                                                                |
| CFBundleIconFiles                                               | Application 의 분류에 따라 적용되는 아이콘 파일 이름을 배열로 표시.                                                                                                          |
| LSRequiresIPhoneOS<br/>(Application requires iOS environment)   | iOS 에서만 동작하는 Bundle 인지 명시함.<br/>기본값은 True 이고 바꾸는 일은 거의 없다.                                                                                            |
| UIRequiredDeviceCapabilities                                    | iTunes/App Store 에 기기 특성 중 앱이 요구하는 특성을 Dictionary 타입으로 명시.                                                                                            |

아래 표에 나타낸 요소들은 iOS 앱에서 반드시 필요한 설정이다. 모두 기본값이 존재하기 때문에 임의로 지정해주지 않아도 괜찮다.

| Name                                   | Description                                                                                                                                                        |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NSMainNibFile(Main nib file base name) | Application main nib 파일 이름을 설정. 커스텀 nib 파일이 존재하는 경우 설정 필요.                                                                                                         |
| UIStatusBarStyle                       | Application 실행 중 status bar 의 스타일 설정. 기본값은 UIStatusBarStyleDefault.                                                                                                |
| UIStatusBarHidden                      | Application status bar 를 실행 중 숨길지 여부 설정. 기본 값은 false.                                                                                                              |
| UIInterfaceOrientation                 | Application 최초 방향 설정. 기본값은 UIInterfaceOrientationPortrait(수직방향).                                                                                                   |
| UIPrerenderedIcon                      | Application Icon 이 Effect 를 가지고 렌더링 되어야 하는지 설정. 기본값은 false.                                                                                                        |
| UIRequiresPersistentWiFi               | Application 이 시스템에게 Wi-Fi 네트워크를 사용하는지 여부를 알려주기 위해 설정.<br/>만약 Wi-Fi 설정이 되어 있지 않으면 네트워크 Dialog 창을 호출한다.(사실확인필요)<br/>30분이 지나면 전력소모를 줄이기 위해 Wi-Fi 연결을 해제함. 기본값은 false. |
| UILaunchImageFile                      | Application Launch Image File Name 을 설정.                                                                                                                           |

**Application Icon and Launch Images**

모든 앱에는 Launch Image 가 기본으로 제공되어야 한다. 모든 앱에는 App Store 와 기기에서 보여주어야 할 Icon 을 제공해야 한다. 앱 Icon 은 크기 등에 따라 여러 버전이 제공될 수 있다.

**Resources in an iOS Application**

iOS Application Bundle 에는 다국어 적용이 되지 않은 Resources 들이 가장 상위 레벨 위치에 위치하게 된다. Executable, Info.plist, App Icon, Launch Image, Nib files 등이 그러하다. 다국어 적용이 되지 않은 파일들을 최상위 디렉토리에 위치시킴과 동시에 하위 디렉토리에 새로운 Resources 들을 정의할 수 있다.

예시로 아래와 같이 다국어 버전과 다국어 버전이 아닌 Resources 를 포함한 Application Bundle 구조를 정의할 수 있다.

> MyApp.app
> - Info.plist
> - MyApp
> - Default.png
> - Icon.png
> - Hand.png
> - MainWindow.nib
> - MyAppViewController.nib
> - WaterSounds/
>   - Water1.aiff
>   - Water2.aiff
> - en.lproj/
>   - CustomView.nib
>   - bird.png
>   - Bye.txt
>   - Localizable.strings
> - jp.lproj/
>   - CustomView.nib
>   - bird.png
>   - Bye.txt
>   - Localizable.strings

## Summary

실행 가능한 코드와 관련 리소스를 한 공간에 묶는 디렉토리 모음이다.

포함하는 요소는 다음과 같다.

* Info.plist = 앱의 설정 파일
* Executable = 앱의 실행 파일
* Resource files = Image, Sound file 등 Executable 을 제외한 파일
* Other support files = 그 외 특별한 목적을 위해 만들어진 파일들.

References:
* [Apple Documentation Archive - Bundle Structure](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW1)
* [thoonk's record - [iOS] App Bundle](https://thoonk.tistory.com/64)