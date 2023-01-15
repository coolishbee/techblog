## Unity for iOS 에서 UIApplicationDelegate Override 하는 법



유니티용 iOS 라이브러리를 개발하다보면 최초 앱이 시작단계에서 **아래와 같은 함수**를 재정의 하여 사용자 정의해야 한다.

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary<UIApplicationLaunchOptionsKey, id> *)launchOptions API_AVAILABLE(ios(3.0));
```



Xcode 를 통해 .m 파일 하나만 만들면 된다.(헤더파일 필요없음)



OverrideUnityAppDelegate.m

```
#import <Foundation/Foundation.h>
#import "UnityAppController.h"

@interface OverrideUnityAppDelegate : UnityAppController
@end

IMPL_APP_CONTROLLER_SUBCLASS(OverrideUnityAppDelegate)

@implementation OverrideUnityAppDelegate

-(BOOL)application:(UIApplication*) application didFinishLaunchingWithOptions:(NSDictionary*) options
{
    NSLog(@"[OverrideUnityAppDelegate application:%@ didFinishLaunchingWithOptions:%@]", application, options);
    return [super application:application didFinishLaunchingWithOptions:options];
}

@end
```



#### Unity Directory Path

`Assets/Plugins/iOS` 소스파일은 이 디렉토리 하위로 두면 된다.

넣고 빌드하면 로그가 찍힐 것이다.



> 출처
>
> http://blog.eppz.eu/override-app-delegate-unity-ios-macos-1/