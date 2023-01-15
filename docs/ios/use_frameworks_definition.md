# use_frameworks! 정의

CocoaPods에 정적 라이브러리 대신 프레임 워크를 사용하려고 함을 알립니다. 

Swift는 Static Libraries를 지원하지 않으므로 프레임 워크를 사용해야합니다.



## 정적 라이브러리와 프레임 워크의 차이점

### Cocoa Touch Frameworks

항상 오픈 소스이며 앱처럼 빌드됩니다. (따라서 Xcode는 앱을 실행할 때와 항상 프로젝트를 정리 한 후에이를 컴파일합니다.) 프레임 워크는 iOS 8 이상 만 지원하지만 프레임 워크에서 Swift 및 Objective-C를 사용할 수 있습니다.

### Cocoa Touch Static Libraries

이름에서 알 수 있듯이 정적입니다. 따라서 프로젝트로 가져올 때 이미 컴파일 된 것입니다. 코드를 보여주지 않고도 다른 사람과 공유 할 수 있습니다. 현재 정적 라이브러리는 Swift를 지원하지 않습니다. 라이브러리 내에서 Objective-C를 사용해야합니다. 앱 자체는 여전히 Swift로 작성할 수 있습니다.



> 출처
>
> https://stackoverflow.com/questions/41210249/why-do-we-use-use-frameworks-in-cocoapods