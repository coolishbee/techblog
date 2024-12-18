---
comments: true
---

게임펍은 퍼블리싱 사업이 주된 사업으로 다양한 개발사와 협업하여 게임을 출시합니다.<br>
이에 따라 반복적인 기능들을 필요로 했고 나아가 사용자 경험을 쌓아 수익창출에 도움이 될 수 있는 SDK 가 필요했습니다.

## 개요

기존에 일회성으로 만들어 사용하던 Unity SDK 이 있었으나 각 써드파티들에 대한 의존성, 확장성, 호환성 이슈들이 예측됐고
c# 으로만 개발된 SDK 였기 때문에 네이티브 기능도 사용할 수 가 없었습니다.<br>
그래서 각각 iOS 와 Android 용 네이티브 SDK 를 개발하여 C# 인터페이스를 통해 Unity SDK 를 제공하기로 결정했습니다.<br>
현재 SDK 팀이 앞으로 어떻게 될지 모르겠지만 향후 언리얼 SDK 도 개발 가능하다는 이점을 가져가는 것도 목표중 하나였습니다.

기본적인 목표는 다른 유니티 SDK 와 같이 C# Interface로 Android, iOS 간에 차이 없이 사용 가능하게 제공하는 거였고
이를 위해 구성한 SDK 구조는 다음과 같습니다.

![sdk-diagram](../img/Gamepub-SDK.png)

## Unity SDK

Unity에서는 그동안 계속해서 [네이티브 플러그인](https://docs.unity3d.com/kr/2021.3/Manual/Plugins.html)에 대한 지원이 발전되고 있었습니다. 이것을 이용해서 Native SDK 를 래핑하는 Unity용 Android Bridge 프로젝트를 만들었습니다.<br>
이렇게 했을 때 얻을 수 있는 장점은 사용자 API 인터페이스만 바뀌지 않는다면 네이티브 SDK 구현부 배포를 원격으로 할 수 있다는 점입니다.

### API 요청 방식

* iOS : Unity에서 제공되는 [iOS용 플러그인 빌드](https://docs.unity3d.com/kr/2021.3/Manual/PluginsForIOS.html)방식을 사용하여 각 네이티브 메서드를 호출합니다.
* Android : Unity에서 제공되는 [AndroidJavaObject](https://docs.unity3d.com/ScriptReference/AndroidJavaObject.html)를 사용하여 각 네이티브 메서드를 호출합니다.

=== "iOS interface"
    ```c#
    [DllImport("__Internal")]
    private static extern void login(string identifier);
    public static void Login(string identifier) {
        return login(identifier);
    }
    ```
=== "Android interface"
    ```c#
    public static void Login(string identifier) {
        var androidObject = new AndroidJavaObject("com.gamepubcorp.sdk.WrapperClass");

        object[] param = new object[1];
        param[0] = identifier;

        return androidObject.Call("login", param);
    }
    ```

### API 응답 방식

네트워크 통신이 있는 API 경우엔 비동기 콜백처리가 필요하므로 네이티브 쪽에서 Unity의 메서드를 호출할 수 있어야 합니다.
Unity에서는 Android와 iOS 마찬가지로 UnitySendMessage 메서드를 제공합니다.

UnitySendMessage 의 정의는 다음과 같습니다.

```
UnitySendMessage("GameObjectName1", "MethodName1", "Message to send");
```

* 호출할 대상의 GameObject의 이름
* 호출할 스크립트의 메서드
* 메서드 파라미터 문자열

이 메서드를 이용해서 토큰을 식별자로 사용하고, 데이터는 JSON 직렬화 파라미터를 사용해서 전달합니다.
여기서 전달된 데이터는 Unity 엔진 런타임의 생성된 Gamepub 객체(GameObject)에서 수신합니다.
수신하는 GameObject 의 종류는 성공, 실패로 나뉘며 직렬화된 데이터를 받아 역직렬화하여 객체로 캐스팅합니다.

Unity에서 이 콜을 받으려면

1. MonoBehaviour 을 상속받는 객체여야 합니다.
2. 그 객체(GameObject)는 Unity 엔진 런타임에 생성되어 있어야 합니다.
3. 해당 스크립트내에 MethodName이란 이름의 메서드가 정의되어 있어야 합니다.

식별자를 사용해서 네이티브 API를 호출하고, 동일한 식별자를 UnitySendMessage 메서드를 통해 반환하면 비동기 작업을 호출한 주체를 식별할 수 있습니다.

=== "iOS"
    ```
    UnitySendMessage("NativeListener", "CallbackMethod", "json with identifier");   
    ```
=== "Android"
    ```
    UnityPlayer.UnitySendMessage("NativeListener", "CallbackMethod", "json with identifier")
    ```
=== "Unity"
    ```c#
    void CallbackMethod(string json) {
        // identifier가 포함된 json 데이터 역직렬화
    }
    ```

전달받은 identifier를 확인하면 어떤 API가 호출했는지 알 수 있기 때문에 하나의 메소드 정의로 여러 다수의 API를 처리하는 장점을 갖습니다.<br>
이렇게 이러한 방법을 통해 네이티브와 Unity C# 간에 데이터를 주고 받습니다.

## Android SDK

첫 번째 버전은 자바로 개발했었고 출시 이후 조금씩 코틀린으로 전환해나갔습니다. 다행히 자바와 코틀린은 한 프로젝트 내에서도 문제없이
작동하였고 추후 새로운 코틀린 프로젝트를 동일한 maven repository에 업로드하여 사용할 수 있어서 생각보다 쉽게 전환할 수 있었습니다.
그리고 초기에는 [jcenter](../android/jcenter.md)를 저장소로 사용했었지만 서비스종료됨에 따라 maven으로 이관했었습니다.

### 의존성 관리

Unity는 공식적으로 Gradle 빌드 시스템을 지원하기 때문에 네이티브 SDK와의 의존성 문제는 아주 쉽게 해결할 수 있었습니다.

### 소셜 로그인 및 인증

![auth](../img/Auth.png)

### 결제

![iap](../img/IAP.png)

결제 모듈 통합

* 구글스토어
* 원스토어
* 갤럭시스토어

안드로이드 경우엔 스토어가 3개였고 모두 라이브러리에 포함되야 했습니다. 그래서 컴파일시 분기처리하여 런타임시 하나의 스토어 인터페이스만 생성되도록 설계했습니다.

### 푸시(FCM)

#### Firebase 인증

Firebase 제품군을 사용하기 위해서는 `google_services.json`의 값을 액세스하는 [Google 서비스 Gradle 플러그인](https://developers.google.com/android/guides/google-services-plugin?hl=ko)이 필요합니다. 이 플러그인은 자바코드로 읽을 수 있는 `xml`형태로 변환되며 유니티내에서는 Firebase SDK 모듈내에서 그 역할을 해줍니다.

그러나 GamepubSDK 내에서 이미 FCM모듈을 포함하고 있었기 때문에 Firebase SDK for Unity 의존성 없이 `google_services.json`을 액세스할 방법을 찾아보았습니다. 그래서 여러 테스트 끝에 [파일 변환](https://dandar3.github.io/android/google-services-json-to-xml.html)후 `Assets/Plugins/Android/FirebaseApp.androidlib/res/values`에 위치하면 컴파일시 읽을 수 있었습니다.

#### 푸시 아이콘 커스텀

푸시 알림 아이콘 설정도 동일한 방법으로 설정이 가능합니다. [Noti Icon generator](http://romannurik.github.io/AndroidAssetStudio/icons-notification.html#source.type=clipart&source.clipart=ac_unit&source.space.trim=1&source.space.pad=0&name=ic_stat_ic_notification)를 이용해서 사이즈별 파일을 `Assets/Plugins/Android/FirebaseApp.androidlib/res/drawable-...` 경로에 추가하면 됩니다.
그리고 `Assets/Plugins/Android/AndroidManifest.xml`에 아래 내용을 추가해야 합니다.
```
...
<application
    ...
    <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/ic_stat_ic_notification" />
</application>
...
```

#### 푸시알림 설정

FCM에서는 2가지 유형의 메시지를 클라이언트로 보낼 수 있습니다.

* Notification : 종종 '표시 메시지'로 간주됩니다. FCM SDK에서 자동으로 처리합니다.
* Data : 클라이언트 앱에서 처리합니다.

보통 서버에선 Notification 타입으로 푸시메시지를 보내곤 하는데 그렇게 되면 클라이언트가 백그라운드에서 수신시 헤드업 푸시알림을 표시하지 않고
작업 표시줄에만 아이콘으로 표시합니다. 그래서 항상 헤드업 푸시알림을 표시하고 싶다면 서버에서 data 타입으로 메시지를 보내야 합니다.

[공식 문서](https://firebase.google.com/docs/cloud-messaging/android/receive?hl=ko#handling_messages)를 참고하면 이렇습니다.

포그라운드, 백그라운드 모두 헤드업 푸시알림을 하려면

* 알림 채널과 [중요도 설정](https://developer.android.com/training/notify-user/channels?hl=ko#importance)(IMPORTANCE_HIGH)
* 서버에서 data 타입으로 메시지 전송

이 두가지가 중요합니다.

그리고 채널의 용도는 다음과 같습니다.

* API 레벨 26 (Android 8 Oreo) 이상의 기기는 로컬 알림을 발송하는 하나의 앱 안에서 여러 개의 채널을 설정하여 채널별로 메시지를 보낼 수 있다.


알림(notification) 메시지 예시
```
{
  "message":{
    "token":"bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
    "notification":{
      "title":"Portugal vs. Denmark",
      "body":"great match!" 
    }
  }
}
```

데이터(data) 메시지 예시
```
{
  "message":{
    "token":"bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
    "data":{
      "Nick" : "Mario",
      "body" : "great match!",
      "Room" : "PortugalVSDenmark" 
    }
  }
}
```

### 네트워크 통신(Retrofit2)

초기버전인 자바를 사용할 때는 콜백지옥 방식을 썼지만 코틀린 프로젝트에서는 Coroutine을 활용하여 비동기 코드를 최대한 단순화시켰습니다. 이 과정에서 APIService를 deferred로 구현했다가 suspend로 바꾸기도 하였습니다.
역시 이쪽에 전문가는 아니라서 공을 드리긴 했지만 여전히 부족하다고 느꼈습니다.

최종 API 디자인

```
├── network
│   ├── ApiProvider.kt
│   ├── ApiService.kt
│   ├── HttpClient.kt
│   ├── reqBody
│   └── respBody
```

```
interface ApiService {
    @POST("/login/{projectId}")
    suspend fun login(@Path("projectId") projectId: Int,
                      @Body data: ReqLogin): Response<RespLogin>
    ...
```
HttpClient.kt :
```
fun login(reqBody: ReqLogin,
          onSuccess: ((RespLogin) -> Unit)?,
          onFailed: ((PubSdkError) -> Unit)?,
          onError: ((Throwable) -> Unit)?
) {
    CoroutineScope(Dispatchers.IO).launch {
        try {
            val response = ApiProvider.provideApi().login(
                projectId,
                reqBody
            )
            if(response.isSuccessful){
                response.body()?.let {
                    if (it.code == PubSdkErrorCode.SUCCESS) {
                        onSuccess?.invoke(it)
                    }else{
                        onFailed?.invoke(PubSdkError(it.code, it.msg))
                    }
                }
            }
        } catch (t: Throwable) {
            onError?.let {
                onError(t)
            }
        }
    }
}
```
Usage :
```
val reqLogin = ReqLogin.createReqLogin(socialProfile)
HttpClient.login(reqLogin,
    onSuccess = {
        // it. (Resp Login)
    },
    onFailed = {
        // it. (API Error)
    },
    onError = {
        // it. (Network Error)
    }
)
```

### Unity에서 연동

Unity 안드로이드에서 빌드시 두가지 의존성 관리방법을 제공합니다.<br>

1. gradle을 이용한 원격 저장소 사용
2. aar파일 직접 사용

그리고 gradle의 버전은 유니티 안에 내장된 gradle 버전에 따라야 하며 현재까지는 6.1.1까지 지원된 상태이고 조만간 7.x가 지원될 것 같습니다.
Unity 버전에 따라 [Unity 안에 내장된 gradle 버전](https://docs.unity3d.com/kr/2023.2/Manual/android-gradle-overview.html)이 다르므로 함부로 gradle의 최신 버전을 사용해선 안됩니다.

* 커스텀 Gradle 템플릿
```
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.android.tools.build:gradle:4.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.61"
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
 
    implementation 'io.github.gamepubcorp:pubsdk:2.1.10' 
    implementation 'com.google.code.gson:gson:2.8.5'
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.61'
    ...
}
```
* Resolver
```
//Dependencies.xml
<dependencies>    
    <androidPackages>
        <repositories>
            ...
        </repositories>
        <androidPackage spec="io.github.gamepubcorp:pubsdk:2.1.10"/>
        <androidPackage spec="com.google.code.gson:gson:2.8.5"/>
        <androidPackage spec="org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.61"/>
    </androidPackages>
</dependencies>
```

### 배포

네이티브 SDK가 Unity용 Android Bridge 프로젝트를 거쳐 Unity 와 통신하기 때문에 네이티브 SDK 는 maven 에 배포하여 Unity용 브릿지 프로젝트내에서 랩핑하여 사용합니다.<br>
그리고 Unity용 Android Bridge 프로젝트는 aar 로 빌드되어 유니티 패키지를 통해 배포됩니다.

#### 배포 순서

* [네이티브 SDK maven 배포](../android/maven-deploy.md)
* Unity용 Android Bridge 빌드 Export
* SDK Unity 프로젝트에 import
* SDK Unity Package export

#### 배포 구조

![GamepubSDK-Android-Deploy](../img/GamepubSDK-Android-Deploy.png)


## iOS SDK

Android SDK와 마찬가지로 Objective-C로 개발해서 서비스 론칭했었고 향후 빠른 대응을 위해 별도로 Swift SDK 프로젝트를 진행했습니다.

그리고 Unity iOS에서 SDK를 연동하려면 브릿지 프로젝트는 Objective-C++로 작성되어야 하며 네이티브 SDK를 Swift로 구현하려면 Objective-C 와 Swift 간에 통신도 해결되야 합니다.


### 의존성 관리

처음에는 Android SDK 처럼 원격 저장소를 이용해서 배포하려고 CocoaPods와 Carthage를 고려했었습니다. [SPM](https://github.com/apple/swift-package-manager)은 출시된지 얼마되지 않았고 여전히 [이슈](https://github.com/apple/swift-package-manager/issues)가 많기 때문에 고려대상이 아니었습니다. 

그래서 CocoaPods와 Carthage 중 유니티 개발자들이 주로 사용하는 의존성 관리도구인 [Resolver](https://github.com/googlesamples/unity-jar-resolver)에서 CocoaPods을 제공하기 때문에 CocoaPods에 배포하기로 했습니다.<br>
우선 Cocoapods에 라이브러리를 배포하려면 [Podspec](https://guides.cocoapods.org/syntax/podspec.html)에 대해 알아야 하고
private 과 public 배포 방식이 달랐기 때문에 두 가지 중 선택해야 했습니다.

정확하게 어떤 것이 합리적인지 알 수 없다면 늘 해왔듯이 두 가지 방법이 있으면 두 가지를 다 해보고 세 가지가 있다면 세 가지를 다 해보고 결정을 했습니다.
물론 시간이 더 걸릴지 몰라도 나중에 다시 돌아갔을때 더 쉽게 전환이 가능하기 때문입니다.

참! 초기에는 Firebase Auth와 FCM을 사용했었습니다. 그렇기 때문에 Cocoapods에 라이브러리 배포시 의존성 문제가 발생했었습니다.
현재는 직접 서버에서 인증처리를 하고 FCM 대신 APNS 를 쓰지만 그 당시에는 [이렇게 해결](https://github.com/coolishbee/cocoapod-google-firebase-dependency-solution)을 했었습니다.
문제는 Firebase 모듈에서는 ios 시뮬레이터용 아키텍쳐를 제공해주지 않아서 였고 그래서 Spec 스크립트단에서 제외시켜주는 옵션을 추가해서
해결했습니다.

```
Pod::Spec.new do |spec|    
  spec.module_name         = "PubSDK"
  ...
  spec.source           = { :git => "https://github.com/../pub-sdk-ios.git", :tag => "#{spec.version}" }  
  spec.source           = { :http => 'https://github.com/../releases/download/0.3.29/PubSDK-0.3.29.zip' }
  ...
end
```

그리고 위와 같이 podspec에서 source에 대한 검증을 git repo를 통해 하는데 회사 소스 주소는 private이기 때문에 사용할 수 없었습니다.
그래서 알고 있던 오픈소스들은 어떻게 할까 싶어서 podspec 파일을 하나하나 열어 보던 중 크래시리포트에 관심이 있어서 분석하던 [PLCrashReporter](https://github.com/microsoft/plcrashreporter)는 좀 다르게 되어 있었습니다.
라이브러리를 zip파일로 압축해서 배포하고 그 zip링크만 http 프로토콜로 연결하고 있었습니다. 여기서 힌트를 얻어
소스없이 framework 만 zip로 압축해서 배포하는 git public 저장소를 만들었고 그 저장소의 zip링크를 활용하여 pod spec lint 유효성체크에
통과했습니다.

하지만 최종 결과적으로는 Cocoapods 을 사용하지 않기로 했습니다. 이유는 다른 써드파티와의 호환성 문제가 있기 때문입니다. 간략하게 설명하자면
회사에서 퍼블리싱하는 프로젝트들은 고정적으로 Firebase, Adjust SDK을 사용하는데 이 SDK들은 Cocoapods을 지원하지만 동적 연결보다는 정적 연결을
더 권장하고 있고 이로인해 많은 유니티 개발자들이 iOS 빌드에 관해 트러블 슈팅을 겪는 점을 알게 되었습니다.
실제로 몇가지 빌드 테스트를 거쳐봤을때 호환성 문제로 Gamepub SDK를 안정적으로 제공하기는 어렵다고 판단했고 프레임워크 파일형태로 제공하기로 결정했습니다.

결국 이러한 이유들로

1. Gamepub iOS SDK내에서는 구글, 페북 라이브러리를 동적 연결로 사용하고 있었는데 정적 연결로 지원해야 했다.
2. 이 중 Google Sign In iOS SDK 는 공식적으로 Carthage 를 지원하지 않고 있었다.
2. 유니티내에서 프레임워크 파일을 바이너리로 추가했을시 다른 써드파티와 빌드 문제가 발생하지 않았다.

안정적인 방법 택했습니다.

### 푸시(APNS)

APNS을 연동하는 네이티브 SDK의 역할은 DeviceToken을 발급받아서 서버로 넘겨주는 일입니다. 보통 iOS 앱에서 DeviceToken을 발급받기 위해
AppDelegate 생명주기의 프로토콜들을 사용하는데 유니티는 자체적으로 AppDelegate를 포함하고 있기때문에
[Unity XCode 프로젝트 구조](https://docs.unity3d.com/kr/2018.4/Manual/StructureOfXcodeProject.html)에 대해 알아야 합니다.

그래서 저는 Unity용 iOS Bridge 프로젝트에서 UnityAppController를 파생하여 새로운 AppDelegate를 만들고 거기서 구현하였습니다.

### 네트워크 통신

Objective-C 버전에선 [AFNetworking](https://github.com/coolishbee/AFNetworkingExample)을 사용했었고
Swift 버전에서는 [Alamofire](https://github.com/coolishbee/AlamofireExample)을 사용했습니다.<br>
Android SDK와 동일하게 최대한 인터페이스 디자인을 단순화 시키기 위해서 고민하고 여러 오픈소스들을 참고하여 
여러 방식으로 테스트하여 만들었습니다.

그리고 Android 와 달리 로그를 출력해주는 기능이 별도로 없기 때문에 커스텀하여 사용했습니다.

PubHttpRouter.swift :
```
enum PubHttpRouter: URLRequestConvertible {
    case login(_ login: ReqLogin)

    private var method: HTTPMethod {
        return .post
    }

    private var path: String {
        switch self {        
        case .login:
            return String(format: "/login/%d",
                          PubAPIConfiguration.shared.projectID)        
        }
    }

    private var parameters: Parameters? {
        switch self {        
        case .login(let login):
            do{
                return try login.encode()
            }catch{
                return nil
            }        
        }
    }

    func asURLRequest() throws -> URLRequest {        
        let strUrl = String(format: "%@%@", Constants.LiveServer.baseURL, path)
        let url = try strUrl.asURL()
        
        var urlRequest = URLRequest(url: url)
        
        urlRequest.method = method
        urlRequest.addValue(ContentType.json.rawValue,
                            forHTTPHeaderField: HTTPHeaderField.contentType.rawValue)        
        
        if let parameters = parameters {
            do {
                urlRequest.httpBody = try JSONSerialization.data(withJSONObject: parameters, options: [])
            } catch {
                throw AFError.parameterEncodingFailed(reason: .jsonEncodingFailed(error: error))
            }
        }
        
        return urlRequest
    }
```

PubHttpClient.swift :
```
public enum PubHttpClient {
    static let session: Session = {
        let configuration = URLSessionConfiguration.af.default
        let apiLogger = PubHttpLogger()
        return Session(configuration: configuration, eventMonitors: [apiLogger])
    }()
    
    @discardableResult
    private static func performRequest<T:Decodable>(_ route:PubHttpRouter,
                                                    decoder: JSONDecoder = JSONDecoder(),
                                                    _ completion:@escaping (Result<T, AFError>)->Void) -> DataRequest {
        return session.request(route).responseDecodable(decoder: decoder) { (response: DataResponse<T, AFError>) in
            completion(response.result)
        }
    }

    public static func login(_ login: ReqLogin,
                             completion:@escaping (Result<RespLogin, AFError>)->Void) {
        performRequest(PubHttpRouter.login(login), completion)
    }
}
```

Usage :
```
let reqLogin = ReqLogin(profile: socialProfile)

PubHttpClient.login(reqLogin) { result in
    switch result {
    case .success(let respLogin):
        break
    case .failure(let error):        
        break
    }
}
```

### Unity에서 연동

Unity iOS에서 Framework를 추가하는 방법은 Unity Editor에서 추가하거나 Unity의 [Xcode API](https://docs.unity3d.com/ScriptReference/iOS.Xcode.Extensions.PBXProjectExtensions.html)를 사용해서 추가할 수 있습니다.

그리고 Xcode에서 설정할 수 있는 인앱결제, 푸시 등을 [ProjectCapabilityManager](https://docs.unity3d.com/ScriptReference/iOS.Xcode.ProjectCapabilityManager.html)로 활성화할 수 있고 파라미터로 받아야 할 설정값들을 [PBXProject](https://docs.unity3d.com/ScriptReference/iOS.Xcode.PBXProject.html)을 사용해 설정할 수 있습니다.

### 배포

#### 배포 순서

* fastlane 으로 프레임워크 빌드 Export
* SDK Unity 프로젝트에 프레임워크 및 Unity용 Objective-C++ Bridge 소스 import
* SDK Unity Package export

#### 배포 구조

![GamepubSDK-iOS-Deploy](../img/GamepubSDK-iOS-Deploy.png)

### Unit Test

iOS의 경우엔 안드로이드와 달리 Swift로 구현된 API를 Objective-C 에서 호출하려면 제약이 따릅니다.<br>
말하자면 Objective-C 에서 Swift로 개발한 API들이 정상적으로 호출이 가능한지 제대로 작동되는지 체크하려면 Unity iOS 빌드해서
확인하기에는 너무 사이클이 길다는 얘기죠. 물론 완벽한 테스트 환경으로 대체가 가능하진 않지만 문제에 대한 범위를 확실히 줄이는 데에는 도움이 됩니다.

그렇기 때문에 XCTest를 이용해 유닛 테스트를 구현해보았습니다.

!!! Example
    이렇게 하면 Unity로 배포되기 전 미리 테스트를 해볼 수 있습니다.

    === "Objc"
        ![xctest-unit01](../img/xctest-unit01.png)
    
    === "Swift"
        ![xctest-unit02](../img/xctest-unit02.png)




## 안드로이드 애플로그인

!!! note "참고"

    Apple은 지난 6월 WWDC(Worldwide Developers Conference)에서 신제품인 Apple로 로그인(Sign in with Apple)을 발표했습니다. 9월 19일 iOS 13 릴리스를 앞두고 App Store 심사 지침도 업데이트되었습니다. 이제 새로운 애플리케이션은 타사 또는 소셜 로그인 서비스를 사용하려면 Apple로 로그인 옵션을 함께 제공해야 하며, 기존 애플리케이션은 2020년 4월부터 이 규정을 준수해야 합니다. 이번 변경사항은 Apple 개발자 사이트에서 자세히 확인할 수 있습니다.

Apple 로그인은 안드로이드에서의 구현이 까다롭습니다. 

공식적인 라이브러리를 지원하지 않으므로 직접 구현해야 하기 때문입니다.
그런데 게임유저 입장에서는 안드로이드와 iOS를 빈번히 오가며 플레이하는 경우가 많습니다. 그렇기 때문에 안드로이드에서도 Apple 로그인을 연동하는 것이 사용자 경험을 높인다고 생각했습니다.
이미 대형 게임개발사에서는 그렇게 많이 하고 있었고 저희 또한 분명히 필요성을 느꼈습니다.

그래서 제가 최종적으로 Custom Tabs을 사용해서 구현한 안드로이드에서의 Apple 로그인 흐름은 이렇습니다.

### 안드로이드 Apple 로그인 Flow

``` mermaid
sequenceDiagram
    autonumber
    actor A as Unity Client
    participant B as Android
    participant C as AppleID Service
    participant D as User Redirect Server

    A->>B: Apple Login Request    
    B->>C: Authenticate User
    C->>D: Login Info Result(POST Method)
    activate D
    D->>D: POST Body Redirect
    deactivate D
    D->>B: Login Info(onNewIntent)
    activate B
    B->>B: parseUri
    deactivate B
    B->>A: Login Result
```
<br>
안드로이드에서 Apple 로그인을 구현하려면 Apple 인증서버에서 보내주는 인증정보를 수신하고 Redirect해줄 서버가 필요합니다.

### WebView 구현

기본적으로 Apple 로그인은 웹 환경을 지원하기 때문에 웹뷰로 구현할 수 있습니다.
여기서 핵심은 백엔드 서버(Redirect 서버)에서 넘겨주는 정보를 어떻게 받을지가 관건이었습니다.

일단 WebView 콜백 메소드로 데이터를 받아야 하는데 이 중에서 onPageFinished 대신 shouldOverrideUrlLoading로 처리한 이유는 이렇습니다.
onPageFinished에 비해 매번 호출되지 않았고 서버에서 Redirect시 정확하게 호출되는 것을 확인했기 때문입니다.

```
@Override
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {    
    return isUrlOverridden(view, request.getUrl());    
}
```

이렇게 shouldOverrideUrlLoading 메소드를 통해서 인증정보를 파싱하여 처리했고 최근까지도 이렇게 사용하고 있습니다.
하지만 얼마전부터 구글에서는 웹뷰로 인증하는 행위에 대해 권장하지 않는다 라는 경고메시지를 안내해주고 있습니다.<br>
그래서 Custom Tabs으로 전환하기 위해 몇가지 사항을 고려했습니다.

1. Custom Tabs가 지원되지 않는 디바이스나 앱플레이어에 대해서 어떻게 대체할 것인가
2. 서버로부터 리다이렉트 수신을 받을 수 있는가
3. 새로운 액티비티 위에 Custom Tabs를 올릴 수 있는가

### Custom Tabs 구현

이미 WebView 이용하여 구현해보았기 때문에 WebView의 shouldOverrideUrlLoading가 하는 역할만 만들어주면 되겠다고 생각했습니다.

구글링을 통하여 방법을 찾던 중에 [이 사이트](https://joebirch.co/android/oauth-on-android-with-custom-tabs/)에서 힌트를 얻었습니다. 
shouldOverrideUrlLoading 메소드는 http프로토콜이어야지만 호출이 되는 특성이 있었는데 그와 달리 스키마 프로토콜은 http나 https을 사용할 수 없고 
고유한 스키마명이어야 했습니다. 
딥링크 방식 자체가 브라우저로 동작하는 것이기 때문에 브라우저단에서 단순 데이터 스트링으로 판단하는 것이 아니라 url주소로 판단합니다.
그래서 수신하는 서버가 없다면 DNS 주소를 찾을 수 없다고 오류가 납니다.

```
<activity
    android:name=".auth.AppleLoginActivity"    
    android:launchMode="singleTask"

    <!-- for OAuth2.0 redirection deep linking -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.BROWSABLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data
            android:host="callback"
            android:scheme="auth"/>
    </intent-filter>
</activity>
```

그리고 인텐트 Flag 값은 `FLAG_ACTIVITY_SINGLE_TOP`을 설정하여 새로운 태스크를 만들지 않고
기존 태스크위에 올라오도록 하여 성공이나 실패시 완전히 Destory되도록 했습니다.

```
val customTabsIntent = CustomTabsIntent.Builder().build()
customTabsIntent.intent.flags = Intent.FLAG_ACTIVITY_SINGLE_TOP
CustomTabActivityHelper.openCustomTabAppleLogin(
    this,
    customTabsIntent,
    url,
    AppleLoginBrowserFallback())
```

어쨌뜬 이 자료를 참고하여 Custom Tabs로 로그인을 호출하고 리다이렉트에 대한 수신은 onNewIntent을 통해서 받았습니다.

```
override fun onNewIntent(intent: Intent?) {
    if (intent?.data != null) {
        parseUri(intent.data!!)
    }
}
```

추가로 앱플레이어나 특정 디바이스에서 크롬앱이 설치되어있지 않은 경우에는 Custom Tabs사용이 제한되기 때문에 
기본브라우저로 실행하도록 Fallback 처리하였습니다. 이 또한 Firebase나 Facebook처럼 모바일에서 OAuth인증을 
활용하는 라이브러리들은 어떻게 처리하는지 참고했습니다.<br>
Fallback 처리시에는 위에 activity 매니페스트 설정에서 `android:launchMode`을 꼭 `singleTask`로 해줘야 합니다.<br>
그렇지 않으면 `onNewIntent`호출이 되지 않을 수도 있습니다. 보통 Custom Tabs가 사용 가능한 상황에서는 `singleTop`을 쓰지만
그렇게 사용하면 기본 브라우저는 다른 Task이기 때문에 동일한 Activity가 존재하지 않아서 새로 생성되어 `onNewIntent`가 호출되지 않는 상황이 생깁니다.
`onNewIntent`가 호출되는 조건은 이미 생성된 Activity를 재사용될 때 호출됩니다.

```
class AppleLoginBrowserFallback : AppleLoginFallback{
    override fun openUri(activity: Activity?, uri: Uri?) {
        val browserIntent = Intent(Intent.ACTION_VIEW, uri)
        activity?.startActivity(browserIntent)
    }
}
```

## 가이드 문서

처음에는 github wiki도 활용해보고 여러 고민을 해보았지만 우리 회사 특성상 유료제품에 돈을 쓰기 어렵기 때문에 
무료사용이 가능한 gitbook에 대해 알고 나서는 바로 gitbook으로 갈아탔다.<br>
이 gitbook의 장점은 참 많은데 우리 상황에서 가장 매력적인 부분은 돈 안 들이고 커스텀 도메인을 설정할 수 있다는 점이었다.

[Gamepub SDK 가이드 문서](https://docs.igamepub.co.kr/sdk-guide/get-started)

