
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

Unity에서는 그동안 계속해서 [네이티브 플러그인](https://docs.unity3d.com/kr/2021.3/Manual/Plugins.html)에 대한 지원이 발전되고 있었습니다. 이것을 이용해서 Native SDK 와 통신하는
브릿지 프로젝트를 만들었습니다.

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

전달받은 identifier를 확인하면 어떤 API가 호출했는지 알 수 있기 때문에 한 번의 UnitySendMessage 호출로 여러 다수의 API를 처리하는 장점을 갖습니다.<br>
이렇게 이러한 방법을 통해 네이티브와 Unity C# 간에 데이터를 주고 받습니다.

## Android SDK

첫번째 버전은 자바로 개발했었고 출시이후 조금씩 코틀린으로 전환해나갔습니다. 다행히 자바와 코틀린은 한 프로젝트내에서도 문제없이
작동하였고 추후 새로운 코틀린 프로젝트를 동일한 maven repository에 업로드하여 사용할 수 있어서 생각보다 쉽게 전환할 수 있었습니다.

### 결제 모듈 통합

* 구글스토어
* 원스토어
* 갤럭시스토어

안드로이드 경우엔 스토어가 3개였고 모두 라이브러리에 포함되야 했습니다. 그래서 컴파일시 분기처리하여 런타임시 하나의 스토어 인터페이스만 생성되도록 설계했습니다.

### 푸시(FCM)

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

### Unity에서 연동

Unity 안드로이드에서 빌드시 두가지 의존성 관리방법을 제공합니다. gradle의 버전은 유니티 안에 내장된 gradle 버전에 따라야 하며 현재까지는 6.1.1까지 지원된 상태이고 조만간 7.x가 지원될 것 같습니다.

* 커스텀 Gradle 템플릿
```
buildscript {    
    ...
 
    dependencies {
        ...
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

네이티브 SDK가 Unity용 브릿지 프로젝트를 거쳐 Unity 와 통신하기 때문에 네이티브 SDK 는 maven 에 배포하여
Unity용 브릿지 프로젝트내에서 랩핑하여 사용합니다. 
브릿지 프로젝트는 aar 로 빌드되어 유니티 패키지를 통해 배포됩니다.

jenkins, fastlane

### Target API 지원


## iOS SDK

Android SDK와 마찬가지로 Objective-C로 개발해서 서비스 론칭했었고 향후 빠른 대응을 위해 별도로 Swift SDK 프로젝트를 실행했습니다.

Unity iOS에서 SDK를 연동하려면 브릿지 프로젝트는 Objective-C++로 작성되어야 하며 Objective-C 와 Swift 간에 통신도 해결되야 합니다.


### 푸시(APNS)

### 네트워크 통신

### Unity에서 연동

CocoaPods와 Carthage중 유니티개발자들이 주로 사용하는 의존성 관리도구인 Resolver에서 CocoaPods을 제공하기 때문에 CocoaPods에 배포하기로 했습니다.

## 소셜로그인

* 구글, 페북, 애플

## 가이드 문서

gitbook
