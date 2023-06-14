
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
        // json 데이터 직렬화
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

### 결제 모듈 통합

* 구글스토어
* 원스토어
* 갤럭시스토어

안드로이드 경우엔 스토어가 3개였고 모두 라이브러리에 포함되야 했습니다. 그래서 컴파일시 분기처리하여 런타임시 하나의 스토어 인터페이스만 생성되도록 설계했습니다.

### 푸시(FCM)

#### 인증

Firebase 제품군을 사용하기 위해서는 `google_services.json`의 값을 액세스하는 [Google 서비스 Gradle 플러그인](https://developers.google.com/android/guides/google-services-plugin?hl=ko)이 필요합니다. 이 플러그인은 자바코드로 읽을 수 있는 `xml`형태로 변환되며 유니티내에서는 Firebase SDK 모듈내에서 그 역할을 해줍니다.

그러나 GamepubSDK 내에서 이미 FCM모듈을 포함하고 있었기 때문에 Firebase SDK for Unity 의존성 없이 `google_services.json`을 액세스할 방법을 찾아보았습니다. 그래서 제가 찾은 해결책은 [파일 변환](https://dandar3.github.io/android/google-services-json-to-xml.html)후 `Assets/Plugins/Android/FirebaseApp.androidlib/res/values`에 위치하면 컴파일시 읽을 수 있었습니다.

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

* 알림 채널과 중요도 설정(IMPORTANCE_HIGH)
* 서버에서 data 타입으로 메시지 전송

이 두가지가 중요합니다.

그리고 채널의 용도는 다음과 같습니다.

* API 레벨 26 (Android 8 Oreo) 이상의 기기는 로컬 알림을 발송하는 하나의 앱 안에서 여러 개의 채널을 설정하여 채널별로 메시지를 보낼 수 있다.

중요도의 [정의](https://developer.android.com/training/notify-user/channels?hl=ko#importance)
        
* IMPORTANCE_HIGH: 알림음이 울리며 헤드업 알림 표시
* IMPORTANCE_DEFAULT: 알림음이 울리며 상태 표시줄에 아이콘 표시
* IMPORTANCE_LOW: 알림음이 없고 상태 표시줄에 아이콘 표시
* IMPORTANCE_MIN: 알림음이 없고 상태 표시줄에 표시되지 않음


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

#### Target api 31 이상 지원

* target api 31 이상 지원


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
그리고 gradle의 버전은 유니티 안에 내장된 gradle 버전에 따라야 하며 현재까지는 6.1.1까지 지원된 상태이고 조만간 7.x가 지원될 것 같습니다.

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

### Target API 지원


## iOS SDK

Android SDK와 마찬가지로 Objective-C로 개발해서 서비스 론칭했었고 향후 빠른 대응을 위해 별도로 Swift SDK 프로젝트를 실행했습니다.

Unity iOS에서 SDK를 연동하려면 브릿지 프로젝트는 Objective-C++로 작성되어야 하며 Objective-C 와 Swift 간에 통신도 해결되야 합니다.


### 의존성 관리

cocoadpos carthage [spm](https://github.com/apple/swift-package-manager)

### 푸시(APNS)

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

PubHttpLogger.swift :
```
class PubHttpLogger: EventMonitor {
    let queue = DispatchQueue(label: "APIEventLogger")
    
    func requestDidFinish(_ request: Request) {
        
        print((request.request?.httpMethod ?? "") +
              (" --> ") +
              (request.request?.url?.absoluteString ?? "")
        )
        print(request.request?.allHTTPHeaderFields ?? [:])
        print(request.request?.httpBody?.toPrettyPrintedString ?? "")
    }
    
    func request<Value>(_ request: DataRequest,
                        didParseResponse response: DataResponse<Value, AFError>) {
        
        print("\(response.response?.statusCode ?? 0)" +
              (" --> ") +
              (request.request?.url?.absoluteString ?? "")
        )
        print(response.data?.toPrettyPrintedString ?? "")
    }
}

extension Data {
    var toPrettyPrintedString: String? {
        guard let object = try? JSONSerialization.jsonObject(with: self, options: []),
              let data = try? JSONSerialization.data(withJSONObject: object, options: [.prettyPrinted]),
              let prettyPrintedString = NSString(data: data, encoding: String.Encoding.utf8.rawValue) else { return nil }
        return prettyPrintedString as String
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

처음에는 Android SDK 처럼 원격 저장소를 이용해서 배포하려고 CocoaPods와 Carthage를 고려했었습니다.
그래서 CocoaPods와 Carthage중 유니티개발자들이 주로 사용하는 의존성 관리도구인 [Resolver](https://github.com/googlesamples/unity-jar-resolver)에서 CocoaPods을 제공하기 때문에 CocoaPods에 배포하기로 했습니다.<br>
우선 Cocoapods에 라이브러리를 배포하려면 [Podspec](https://guides.cocoapods.org/syntax/podspec.html)에 대해 알아야 하고
private 과 public 배포 방식이 달랐기 때문에 두 가지 중 선택해야 했습니다.

정확하게 어떤 것이 합리적인지 알 수 없다면 늘 해왔듯이 두 가지 방법이 있으면 두 가지를 다 해보고 세 가지가 있다면 세 가지를 다 해보고 결정을 했습니다.
물론 시간이 더 걸릴지 몰라도 나중에 다시 돌아갔을때 더 쉽게 전환이 가능하기 때문입니다.

참! 초기에는 Firebase Auth와 FCM을 사용했었습니다. 그렇기 때문에 Cocoapods에 라이브러리 배포시 의존성 문제가 발생했었습니다.
현재는 직접 서버에서 인증처리를 하고 FCM 대신 APNS 를 쓰지만 그 당시에는 [이렇게 해결](https://github.com/coolishbee/cocoapod-google-firebase-dependency-solution)을 했었습니다.
문제는 Firebase 모듈에서는 ios 시뮬레이터용 아키텍쳐를 제공해주지 않아서 였고 그래서 Spec 스크립트단에서 제외시켜주는 옵션을 추가해서
해결했습니다.

그리고 podspec 에서 source 에 대한 검증을 git repo를 통해 하는데 회사 소스 주소는 private이기 때문에 사용 할 수 없었습니다.
그래서 알고 있던 오픈소스들은 어떻게 할까 싶어서 podspec 파일을 하나하나 열어 보던 중 크래시리포트에 관심이 있어서 분석하던 [PLCrashReporter](https://github.com/microsoft/plcrashreporter)는 좀 다르게 되어 있었습니다.
라이브러리를 zip파일로 압축해서 배포하고 그 zip링크만 http 프로토콜로 연결하고 있었습니다. 여기서 힌트를 얻어
소스없이 framework 만 zip로 압축해서 배포하는 git public 저장소를 만들었고 그 저장소의 zip링크를 활용하여 pod spec lint 유효성체크에
통과했습니다.

하지만 최종 결과적으로는 Cocoapods 을 사용하지 않기로 했습니다.

1. 
2. 
3. 

### 배포

#### 배포 순서

* fastlane 으로 프레임워크 빌드 Export
* SDK Unity 프로젝트에 프레임워크 및 Unity용 Objective-C++ Bridge 소스 import
* SDK Unity Package export

#### 배포 구조

![GamepubSDK-iOS-Deploy](../img/GamepubSDK-iOS-Deploy.png)

## 소셜로그인

* 구글, 페북, 애플

## 가이드 문서

처음에는 github wiki도 활용해보고 여러 고민을 해보았지만 우리 회사 특성상 유료제품에 돈을 쓰기 어렵기 때문에 
무료사용이 가능한 gitbook에 대해 알고 나서는 바로 gitbook으로 갈아탔다.<br>
이 gitbook의 장점은 참 많은데 일단 돈 안 들이고 커스텀 도메인을 설정할 수 있다는 점이 맘에 들었다.