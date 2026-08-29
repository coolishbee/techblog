---
comments: true
---

# Unity 딥링크 트러블슈팅: 젠킨스 빌드에서만 Universal Link가 죽는 이유

## 들어가며

사내 Unity Deeplink 패키지는 그동안 커스텀 스킴(URI 스킴)만 제공했다. 브라우저나 메신저에서 앱이 바로 열리지 않는 경우가 많았고, 마케팅 측에서도 도메인 기반 링크를 원해 Android의 App Link와 iOS의 Universal Link를 추가하게 됐다.

작업 범위는 다음과 같았다.

- Android: App Link와 assetlinks.json 추가
- iOS: Universal Link와 apple-app-site-association(AASA) 추가

도메인 검증 파일을 올릴 테스트 서버를 따로 구축하는 대신, AASA와 assetlinks.json 호스팅을 제공하는 AppsFlyer OneLink로 검증했다.

!!! warning "핵심 결론"
    Universal Link 구현에는 문제가 없었다. Jenkins 파이프라인의 마지막 fastlane resign 단계가 기존 entitlement를 유지하지 못하면서 associated-domains 값을 잃어버린 것이 원인이었다.

## 증상: Xcode 빌드는 되고 Jenkins 빌드는 안 된다

동일한 코드라도 설치 경로에 따라 결과가 갈렸다.

| 테스트 대상 | 설치 또는 빌드 경로 | OneLink 결과 |
| --- | --- | --- |
| 운영 게임 | Jenkins CLI 빌드·배포 IPA | 앱이 열리지 않음 |
| 동일 프로젝트 | 로컬 Xcode 빌드·설치 | 정상 동작 |
| 패키지 테스트 앱 | AppsFlyer OneLink 테스트 | Android와 iOS 모두 정상 동작 |

Xcode GUI로 설치한 앱은 Universal Link가 열렸지만, Jenkins에서 빌드·배포한 IPA만 실패했다. 따라서 코드보다 빌드 파이프라인을 우선 의심했다.

## 추적 과정

### 1. OneLink와 AASA가 유효한지 확인

Universal Link는 iOS가 도메인의 apple-app-site-association 파일을 받아 앱과 도메인을 연결하는 구조다. 먼저 OneLink 도메인의 AASA를 직접 확인하고, [AppsFlyer OneLink 대시보드](https://hq1.appsflyer.com/onelink)에서 템플릿 설정도 점검했다.

앱의 appID(팀 ID와 번들 ID 조합)가 올바르게 포함되어 있었고, Xcode 빌드에서는 실제로 링크가 열렸다. 이 결과는 서버와 OneLink 설정이 정상이라는 강한 근거가 됐다.

### 2. Unity PostBuildProcess에서 entitlement 연결 확인

Universal Link가 동작하려면 Xcode 프로젝트에 Associated Domains capability가 등록되고, entitlements 파일의 applinks 도메인이 CODE_SIGN_ENTITLEMENTS 빌드 설정에 연결되어야 한다.

처음에는 entitlements 파일이 생성되더라도 코드사인에 연결되지 않는 경우를 의심해 다음 설정을 명시적으로 추가했다.

~~~csharp
// CODE_SIGN_ENTITLEMENTS 빌드 설정 명시적 추가
proj.SetBuildProperty(targetGuid, "CODE_SIGN_ENTITLEMENTS", "Unity-iPhone.entitlements");
~~~

기존의 직접 조작 방식은 Unity가 제공하는 ProjectCapabilityManager 기반으로 교체했다.

=== "기존 방식: 직접 조작"

    - entitlements 파일을 직접 읽고 쓴다.
    - associated-domains 중복을 직접 확인한다.
    - CODE_SIGN_ENTITLEMENTS와 pbxproj를 직접 수정한다.

=== "개선 방식: ProjectCapabilityManager"

    - entitlements 파일을 생성·관리한다.
    - 중복 도메인을 방지한다.
    - CODE_SIGN_ENTITLEMENTS와 Capability 등록을 함께 처리한다.
    - 수동으로 유지해야 할 코드량을 줄인다.

!!! tip "확인 기준"
    Xcode에서 내보낸 결과물에 entitlement가 보인다고 해서 배포 IPA까지 보장되지는 않는다. 최종 IPA의 코드사인 정보를 반드시 확인해야 한다.

### 3. 최종 IPA의 코드사인 정보를 직접 검증

IPA는 ZIP 형식이므로 압축을 풀어 Payload 아래의 앱을 얻은 뒤, codesign으로 실제 서명에 포함된 entitlement를 확인할 수 있다.

~~~bash
unzip <ipa-file>
codesign -d --entitlements - Payload/TSPackages.app
~~~

Jenkins에서 배포한 IPA의 associated-domains 값은 구체적인 도메인 배열이 아니라 와일드카드 하나였다.

??? warning "문제가 있는 Jenkins IPA의 entitlement 출력"

    ~~~text
    $ codesign -d --entitlements - TSPackages.app

    [Dict]
        [Key] application-identifier
        [Value]
            [String] {teamID}.com.percent.ios.percentpackages
        [Key] aps-environment
        [Value]
            [String] production
        [Key] com.apple.developer.applesignin
        [Value]
            [Array]
                [String] Default
        [Key] com.apple.developer.associated-domains
        [Value]
            [String] *
    ~~~

비교를 위해 앱스토어에서 내려받은 타사 게임도 같은 방식으로 확인했다.

??? success "정상 앱의 entitlement 출력"

    ~~~text
    $ codesign -d --entitlements - Kingshot.app
    Executable=/Applications/Kingshot.app/Wrapper/kingshot.app/kingshot
    [Dict]
        [Key] application-identifier
        [Value]
            [String] 4V98RYJDSR.com.run.tower.defense
        [Key] aps-environment
        [Value]
            [String] production
        [Key] com.apple.developer.applesignin
        [Value]
            [Array]
                [String] Default
        [Key] com.apple.developer.associated-domains
        [Value]
            [Array]
                [String] applinks:appleunilink.centurygame.com
    ~~~

| 비교 항목 | Jenkins 배포 IPA | 정상 IPA |
| --- | --- | --- |
| associated-domains | 문자열 * | applinks 도메인이 담긴 배열 |
| Universal Link 동작 | 실패 | 정상 |

이로써 Jenkins에서 나온 IPA는 최종 서명 과정에서 associated-domains를 잃어버린다는 사실이 분명해졌다. Universal Link는 Xcode 프로젝트 파일이 아니라 최종 서명에 포함된 entitlement를 기준으로 동작한다.

### 4. Jenkins 빌드 파이프라인의 경계 좁히기

Jenkins의 iOS 빌드는 다음 순서로 진행됐다.

~~~text
Unity 빌드 → Xcode 프로젝트 및 Workspace 추출 → fastlane run build_app → fastlane run resign
~~~

처음에는 프로비저닝 프로파일 유형을 의심했다. Apple Developer 센터에는 ad-hoc과 App Store 프로파일만 있었지만, fastlane 실행 파라미터에는 development 유형이 설정되어 있었다. 존재하지 않는 development 프로파일을 사용하면서 와일드카드 값으로 덮어써졌다는 가설을 세웠다.

!!! info "반증된 가설"
    빌드 명령은 바꾸지 못하는 상황이어서 development 유형의 프로비저닝 프로파일을 새로 만들어 다시 빌드했다. 결과는 같았으므로 프로파일 유형만이 원인은 아니었다.

PostBuildProcess 개선 후에는 리사인 이전 원본 IPA도 다시 확인했다. 이번에는 원본에 올바른 applinks 도메인이 포함돼 있었다.

??? success "리사인 이전 원본 IPA의 entitlement 출력"

    ~~~text
    $ codesign -d --entitlements - TSPackages.app
    Executable=/Users/111percent/Documents/Payload/TSPackages.app/TSPackages
    [Dict]
        [Key] application-identifier
        [Value]
            [String] {teamID}.com.percent.ios.percentpackages
        [Key] com.apple.developer.applesignin
        [Value]
            [Array]
                [String] Default
        [Key] com.apple.developer.associated-domains
        [Value]
            [Array]
                [String] applinks:tslink.onelink.me
    ~~~

따라서 Unity 빌드와 build_app 단계는 정상이며, 문제 구간은 마지막 resign 단계로 좁혀졌다.

### 5. resign 로그에서 원인 확정

fastlane의 resign은 내부적으로 sigh의 resign.sh를 호출한다. 기존 앱의 entitlement를 유지하려면 use_app_entitlements 설정이 적용되어 entitlements 파일을 지정하는 -e 옵션이 호출 명령에 포함되어야 한다.

??? warning "resign 호출 로그"

    ~~~text
    [2026-03-13T07:42:43.216Z] + fastlane run resign
    [2026-03-13T07:42:44.090Z] [16:42:43]: --- Step: resign ---
    [2026-03-13T07:42:44.090Z] /opt/homebrew/Cellar/fastlane/2.232.2/libexec/gems/fastlane-2.232.2/sigh/lib/assets/resign.sh
        build_ios_percentpackages/com.percent.ios.percentpackages_dev_1.3.5-2.ipa
        3498C2EB80897DF74D1927164674552E6B879192
        -p /Users/build/jenkins-remote/workspace/****/ds/ts/core/test/application/build-scripts/includes/provisioningProfiles/adhocpercentpackages.mobileprovision
        build_ios_percentpackages/com.percent.ios.percentpackages_dev_1.3.5-2.ipa
    ~~~

로그에는 프로비저닝 프로파일을 넘기는 -p 옵션만 있고, 기존 entitlement 파일을 지정해야 할 -e 옵션은 없었다. 즉 use_app_entitlements: "true" 설정이 실제 resign 호출에는 반영되지 않았다.

!!! warning "원인 확정"
    entitlement를 지정하지 않은 재서명은 프로비저닝 프로파일 기준으로 다시 서명한다. 이 과정에서 associated-domains가 와일드카드 값으로 덮어써졌고, Universal Link가 동작하지 않았다. Xcode GUI 설치본은 resign 단계를 거치지 않으므로 정상 동작한 것이다.

## 재발 방지 체크리스트

같은 증상이 발생하면 다음 순서로 확인한다.

| 순서 | 확인 항목 | 확인 방법 |
| --- | --- | --- |
| 1 | AASA와 OneLink 템플릿이 유효한가 | AASA 직접 조회, OneLink 대시보드 |
| 2 | Xcode 프로젝트에 entitlement가 생성·연결되는가 | Unity PostBuildProcess, ProjectCapabilityManager |
| 3 | 최종 IPA 서명에 associated-domains가 있는가 | codesign -d --entitlements - |
| 4 | 리사인 전 원본 IPA에도 값이 있는가 | 원본 IPA를 보관한 뒤 같은 명령으로 비교 |
| 5 | resign이 entitlement를 유지하는가 | 로그에서 -e 옵션 유무 확인 |

## 배운 점

- "설정했다"와 "최종 서명에 반영됐다"는 다르다. Universal Link 문제는 Xcode 프로젝트가 아니라 codesign 출력으로 판단해야 한다.
- 파이프라인 문제는 중간 산출물을 끊어 검증하면 빠르게 범위를 좁힐 수 있다. 원본 IPA와 리사인 후 IPA를 비교하면 용의자를 한 단계로 제한할 수 있다.
- fastlane resign을 사용한다면 use_app_entitlements 설정이 실제 호출의 -e 옵션으로 전달되는지 로그에서 확인해야 한다.
- Unity에서 entitlement를 직접 조작하기보다 ProjectCapabilityManager를 사용하면 중복 방지와 CODE_SIGN_ENTITLEMENTS 연결을 함께 관리할 수 있다.

## 참고 자료

- [AppsFlyer OneLink를 이용한 딥링크 구조](https://velog.io/@gudrmsglgl/AppsFlyer-OneLink%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-DeepLinking#-%EB%A7%81%ED%81%AC%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B8%B0%EB%B3%B8-%EA%B5%AC%EC%A1%B0)
- [iOS Universal Link 정리](https://cording-cossk3.tistory.com/308)
- [AppsFlyer OneLink 가이드](https://support.appsflyer.com/hc/en-us/articles/115005248543-OneLink-guide)
- [AppsFlyer iOS 초기 설정](https://dev.appsflyer.com/hc/docs/dl_ios_init_setup)
- [AppsFlyer 통합 딥링크](https://dev.appsflyer.com/hc/docs/unifieddeeplink)
- [AppsFlyer OneLink 템플릿 생성](https://support.appsflyer.com/hc/en-us/articles/207032246-Create-a-OneLink-template)
- [AppsFlyer OneLink 문제 해결과 자주 묻는 질문](https://support.appsflyer.com/hc/en-us/articles/360014821438-OneLink-troubleshooting-and-FAQ)
- [AppsFlyer Unity 플러그인 딥링크 통합](https://github.com/AppsFlyerSDK/appsflyer-unity-plugin/blob/master/docs/DeepLinkIntegrate.md)
