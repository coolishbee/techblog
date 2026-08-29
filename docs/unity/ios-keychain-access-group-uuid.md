---
comments: true
---

# Keychain access group이 바뀌자 UUID가 초기화됐다

## 들어가며

빌드 파이프라인을 교체한 뒤 UUID가 바뀐다는 제보가 들어왔다. 코드 자체는 그대로였고, 달라진 것은 keychain access group 형식뿐이었다.

기존에는 teamid.* 형태를 사용했지만, 이후 teamid.bundleid 형태로 바뀌었다. 그 순간부터 이전에 저장한 UUID를 조회하지 못했고, 앱은 새 UUID를 발급해 저장하고 있었다.

!!! warning "핵심 원인"
    UUID 생성 로직이 아니라 keychain의 저장 위치가 바뀐 것이 문제였다. CI가 암묵적으로 주입하던 와일드카드 그룹을 앱 전용 그룹으로 명시하면서, 기존 UUID는 삭제되지 않은 채 조회 대상에서만 빠졌다.

## 내 소스에는 그 설정이 없었다

access group을 바꾼 기억이 없었다. 그럴 만도 했다. Unity 빌드의 PostBuildProcess에는 keychain-access-groups를 정의하는 코드가 전혀 없었다.

실제로 값을 넣고 있던 주체는 빌드 머신이었다. CI 단계가 entitlements를 임의로 생성·주입하고, 그 안에서 와일드카드 그룹으로 서명하고 있었다. 프로젝트 소스만 봐서는 알 수 없는 블랙박스였다.

그러다 iOS Universal Link를 붙이게 됐다. Associated Domains는 코드사인된 entitlements에 포함돼야 동작하므로, entitlements를 CI에만 맡길 수 없었다. 직접 구성하면서 앱 전용 설정을 추가했다.

결과적으로 없던 설정을 새로 정의한 것이 아니라, CI가 뒤에서 넣어주던 와일드카드 그룹을 앱 전용 그룹으로 교체한 셈이었다.

## access group이 바뀌면 값을 잃는 이유

Keychain 아이템은 access group 단위로 격리된다. 따라서 access group이 바뀌면 이전 그룹에 저장한 아이템은 조회되지 않는다. 코드 입장에서는 저장한 적이 없는 상태와 구분할 수 없으므로, 새 UUID를 조용히 만들어 저장하게 된다.

중요한 점은 access group을 명시하지 않아도 그룹이 없는 것은 아니라는 사실이다. entitlements에 keychain access group을 설정하지 않으면 기본 저장소는 teamid.bundleid가 된다. 즉 설정을 지우는 행위도 저장 그룹을 바꾸는 행위다.

=== "이전: CI가 주입한 설정"

    ~~~xml
    <!-- 프로젝트 소스에는 없고, CI가 빌드 단계에서 생성·주입했다. -->
    <key>keychain-access-groups</key>
    <array>
      <string>$(AppIdentifierPrefix)*</string>
    </array>
    ~~~

=== "이후: 프로젝트가 직접 구성한 설정"

    ~~~xml
    <!-- Universal Link를 추가하며 associated-domains와 앱 전용 그룹을 구성했다. -->
    <key>com.apple.developer.associated-domains</key>
    <array>
      <string>applinks:내-도메인</string>
    </array>
    <key>keychain-access-groups</key>
    <array>
      <string>$(AppIdentifierPrefix)$(CFBundleIdentifier)</string>
    </array>
    ~~~

teamid.* 와일드카드 그룹을 사용했다는 것은 같은 team prefix를 가진 여러 프로젝트가 같은 UUID를 공유했을 가능성도 뜻한다. 이 동작이 의도된 것인지부터 확인해야 한다.

## 대응 전에 확인한 것

원인 가설은 빠르게 세웠지만, 마이그레이션 로직을 작성하기 전에 다음을 검증해야 했다.

| 확인 항목 | 이유 |
| --- | --- |
| UUID가 실제 teamid.* keychain에 저장되는가 | 기존 저장 위치를 확인해야 한다. |
| 다른 프로젝트도 같은 UUID를 사용하는가 | 와일드카드 그룹의 공유 범위를 파악해야 한다. |
| access group을 바꾸면 기본 저장소도 바뀌는가 | 설정 변경의 실제 영향을 확인해야 한다. |
| access group 미설정 시 기본값이 teamid.bundleid인가 | 제거가 안전한지 판단해야 한다. |
| teamid.bundleid와 teamid.*를 함께 등록해 둘 다 접근할 수 있는가 | 마이그레이션 가능 여부가 결정된다. |
| loadUuid가 두 번 호출되는 것이 의도된 것인가 | 상태가 달라질 수 있는 호출 경로를 제거해야 한다. |

마지막 두 항목이 대응 설계를 갈랐다. 두 그룹에 동시에 접근할 수 있다면, 옛 그룹에서 값을 읽어 새 그룹으로 저장하는 마이그레이션이 가능하다.

## 대응은 두 갈래로 나눴다

이미 UUID가 바뀐 프로젝트와, 아직 파이프라인이 바뀌지 않은 프로젝트는 같은 로직으로 처리할 수 없었다.

=== "아직 이슈가 발생하지 않은 프로젝트"

    새 그룹에 UUID가 없으면 옛 그룹을 조회한다. 값이 있으면 새 그룹에 다시 저장하고 그대로 사용한다. 사용자는 UUID 변경을 경험하지 않는다.

=== "이미 이슈가 발생한 프로젝트"

    옛 UUID가 새 값으로 덮였거나 새 그룹의 값이 이미 자리 잡았을 수 있어 복구를 보장할 수 없다. 대신 legacy UUID 변경과 저장 실패를 C# 이벤트로 전달해 발생 규모를 추적한다.

!!! info "복구보다 관측"
    복구할 수 없는 상태를 억지로 되돌리기보다, UUID 변경과 저장 실패를 관측 가능하게 만드는 방향을 선택했다.

## 함께 정리한 구현

### entitlements 생성 주체 일원화

XcodeOption.cs를 제거하고 iOS PostBuildProcess 구조를 개편했다. CI와 프로젝트가 같은 entitlements 파일을 만들면 어느 쪽의 값이 최종 산출물에 반영됐는지 IPA를 열어 보기 전까지 알 수 없다.

access group처럼 데이터 유실로 직결되는 설정에서는 생성 주체를 한 곳으로 모으는 일이 기능 추가보다 먼저다.

### 식별자 조합 보정

Team ID, AppIdentifierPrefix, bundle identifier를 조합해 access group 문자열을 만드는 로직을 보정했다. 세 값의 처리 방식이 어긋나면 존재하지 않는 그룹을 조회하게 된다.

### 네이티브 인터페이스와 호출 경로 단순화

UUID 로드·저장 인터페이스를 단순화하고, 저장 실패 상태가 C#까지 전달되도록 바꿨다. 이전에는 저장 실패가 호출부에 전달되지 않아 원인을 추적할 방법이 없었다.

iOS 네이티브 문자열 전달도 Unity 마샬링 방식으로 정리해 메모리 관리를 개선했다. 또한 loadUuid가 두 번 호출되던 구조를 한 번으로 줄였다. 두 호출 사이에 상태가 바뀔 수 있는 구조는 자체로 불안정하다.

## identifierForVendor는 생각보다 잘 안 바뀐다

패키지는 UUID를 SystemInfo.deviceUniqueIdentifier로 만들고 있었고, iOS에서는 내부적으로 identifierForVendor를 사용한다.

앱을 재설치하면 값이 새로 나올 것으로 예상했지만, 실제로는 같은 값이 나왔다.

| 구분 | 조건 |
| --- | --- |
| 반드시 변경되는 경우 | 같은 벤더 앱을 모두 삭제한 뒤 재설치, Xcode 테스트 빌드 설치, Ad-hoc 배포 설치, App Store 앱의 다른 개발자 계정 이전 |
| 변경될 수 있는 경우 | TestFlight 빌드에서 App Store 빌드로 업데이트, iOS 베타 버전으로 OS 업그레이드 |
| 변경되지 않는 경우 | 같은 벤더의 앱이 하나라도 기기에 남아 있는 동안의 앱 업데이트 또는 정식 OS 업그레이드 |

> UUID 유지의 기준은 같은 벤더의 앱이 기기에 하나라도 설치되어 있는지다. 모든 앱을 삭제하면 다음 설치에서 새 값이 발급된다.

이번에는 Xcode Development로 재설치했음에도 값이 유지됐다. 문서상 Xcode 설치는 값이 바뀌는 조건에 해당하지만, 삭제 없이 덮어쓴 설치였거나 같은 벤더의 다른 앱이 남아 있었는지 추가 확인이 필요하다.

어느 쪽이든 결론은 같다. 이 값은 저장하더라도 언제든 바뀔 수 있다고 가정해야 한다. 값이 바뀌었을 때 앱이 조용히 다른 사용자로 취급하기보다, 변경 사실을 감지하고 처리할 수 있어야 한다. legacy UUID 변경을 이벤트로 올린 이유도 여기에 있다.

## 정리

이번 사고의 원인은 UUID 생성 로직이 아니라 저장 위치를 결정하는 설정이었다. 코드 한 줄을 바꾸지 않았는데 데이터가 사라진 이유도 여기에 있었다.

그리고 그 설정은 프로젝트 소스에 없었다. 설정이 없다는 말은 사용하지 않는다는 뜻이 아니라, 다른 누군가가 대신 넣고 있을 수 있다는 뜻이다. 그 값을 처음 명시하는 순간은 새 설정을 추가하는 시점이 아니라 기존 동작을 덮어쓰는 변경 시점일 수 있다.

다시 같은 문제를 겪는다면 다음 순서로 확인한다.

| 확인 | 이유 |
| --- | --- |
| 최종 빌드 산출물의 entitlements access group 값 | 소스와 최종 산출물이 다를 수 있다. |
| 소스에 없는 설정이 산출물에 포함되는지 | 설정이 없다는 것이 미사용을 뜻하지는 않는다. |
| access group 생성·수정 주체가 한 곳으로 모였는지 | 여러 곳에서 수정하면 추적할 수 없다. |
| 옛 그룹과 새 그룹을 동시에 등록해 읽을 수 있는지 | 마이그레이션 가능 여부가 결정된다. |
| 저장 실패가 호출부에 전달되는지 | 조용한 실패는 원인 추적을 막는다. |
| 값 변경을 감지할 수 있는지 | 복구할 수 없는 상황도 관측할 수 있어야 한다. |

기기 식별자를 영속 값처럼 다루는 코드는 언젠가 한 번은 이 문제를 만날 수 있다.

## 참고 자료

- [Unity iOS 플러그인](https://docs.unity3d.com/2020.1/Documentation/Manual/PluginsForIOS.html)
- [Unity SystemInfo.deviceUniqueIdentifier](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/SystemInfo-deviceUniqueIdentifier.html)
- [Apple UIDevice.identifierForVendor](https://developer.apple.com/documentation/uikit/uidevice/identifierforvendor)

이번 사고의 계기가 된 Universal Link 적용 문제는 [Unity 딥링크 트러블슈팅](unity-deeplink.md)에서 다룬다.
