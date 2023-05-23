
게임펍은 퍼블리싱 사업이 주된 사업으로 다양한 개발사와 협업하여 게임을 출시합니다.<br>
이에 따라 반복적인 기능들을 필요로 했고 나아가 사용자 경험을 쌓아 수익창출에 도움이 될 수 있는 SDK 가 필요했습니다.

## 개요

기존에 일회성으로 만들어 사용하던 Unity SDK 이 있었으나 각 써드파티들에 대한 의존성, 확장성, 호환성 이슈들이 예측됐고
c# 으로만 개발된 SDK 였기 때문에 네이티브 기능도 사용할 수 가 없었습니다.<br>
그래서 저는 각각 iOS 와 Android 용 네이티브 SDK 를 개발하여 C# 인터페이스를 통해 Unity SDK 를 제공하기로 결정했습니다.
현재 SDK 팀이 앞으로 어떻게 될지 모르겠지만 향후 언리얼 SDK 도 개발 가능하다는 이점을 가져가는 것도 목표중 하나였습니다.

게임펍 SDK 기본 구성 요소는 다음과 같습니다.

![sdk-diagram]()

## 구현

### Unity용 네이티브 플러그인

Unity에서는 네이티브와 통신할 수 있는 기능을 제공해줍니다. 그 기능을 이용하여 Unity C# 과 각 네이티브(Android, iOS)들과
통신하는 브릿지를 만들어야 합니다.

#### 비동기 작업 처리

iOS는 DllImport를 사용해서 공개된 인터페이스를 Objective-C++에서 Unity C#으로 임포트합니다.
Android는 AndroidJavaObject를 사용해서 SDK 네이티브 메서드를 호출합니다.

=== "iOS interface"
    ```c#
    [DllImport("__Internal")]
    private static extern int foo();
    public static int Foo() {
        return foo();
    }
    ```
=== "Android interface"
    ```c#
    public static int Foo() {
        var androidObject = new AndroidJavaObject("com.gamepubcorp.pubsdk.SomeClass");
        return androidObject.Call<int>("foo");
    }
    ```

네트워크 통신이 있는 API 경우엔 비동기 콜백이 필요하며, 파라미터를 통해 네이티브와 Unity C# 간에 데이터를 주고 받습니다.
비동기 콜백을 처리할때는 네이티브 쪽에서 UnitySendMessage 메서드를 호출합니다.
이 메서드를 이용해서 토큰을 식별자로 사용하고, 데이터는 JSON 직렬화 파라미터를 사용해서 전달합니다.


#### JSON과 Serializable로 데이터 전달

네이티브와 Unity에서 모두 해석할 수 있는 포맷으로 데이터를 직렬화하는 방법이 필요합니다. 이렇게 서로 다른 언어 간에 데이터를 교환해야 할 때는 자연스럽게 JSON을 선택하게 됩니다. JSON은 이해하기 쉽고 네이티브와 Unity 양쪽에서 완벽하게 지원됩니다.

```
// Shared structure for token between native side and Unity.
{
  "identifier": "abcdefg...", // The received GUID from Unity.
  "value": {                  // A nested object represents native object.
    "token": "...",
    ...
  }
}
```


### iOS에서 SDK 통합(Swift)

Objective-C 로 개발했었고 추후 Swift로 전환했습니다.

Swift SDK를 Objective-C에서 사용하기 위해 래퍼(wrapper)를 추가로 준비했습니다. 이 래퍼도 Swift로 작성됐지만 Objective-C와 호환되는 언어 기능만 사용되었습니다. Unity로 작업할 때, 내보내기한 프로젝트가 실제론 Objective-C++ 프로젝트이기 때문에 이 래퍼와 통신해야 합니다.

### Android에서 SDK 통합

#### 커스텀 Gradle 템플릿을 사용해서 의존성 관리

Android 플랫폼용 Unity 프로젝트에서는 Gradle 빌드 시스템을 사용합니다.
Resolver 사용

* buildscript 섹션

```
buildscript {
    ext.kotlin_version = '1.3.11'
    ...
 
    dependencies {
        ...
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
 
    }
}
```

* dependencies 섹션

```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
 
    implementation 'io.github.gamepubcorp:pubsdk:1.0.1'
 
    implementation 'com.google.code.gson:gson:2.8.5'
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
 
    ...
}
```

#### Target API 지원

#### 결제 모듈 통합

