## 설정

[공식 가이드](https://developers.google.com/android-publisher/getting_started#service-account)

1. Google Play Console - 설정 - API 액세스 으로 이동
2. 개발자 계정을 신규 또는 기존 Google Cloud 프로젝트에 연결합니다.
3. [연결된 Google Cloud 프로젝트로 이동해서](https://console.cloud.google.com/apis/library) Google Play Developer API를 사용 설정합니다.
4. Google Play Developer API에 액세스할 수 있는 관련 Google Play Console 권한을 가지는 서비스 계정을 설정합니다.

## Google Receipt Validation
### 서비스 계정 생성 및 설정

1. Google Play Console - 설정 - API 액세스 - 서비스 계정 생성
2. 서비스 계정 - KEYS - ADD KEY - Create new Key - JSON file Download
>json 파일은 소스단에서 인증시 필요합니다.
3. Google Play Console - 설정 - API 액세스 - 서비스 계정 - Play Console 권한 보기 - 서비스 계정에 대한 권한 추가
>Play Console의 앱에 대한 재무 권한으로 액세스 권한 부여
4. **매우 중요**: 변경 사항이 적용되도록 최소 24 시간 동안 기다리십시오.

[참고 사이트](https://stackoverflow.com/questions/43536904/google-play-developer-api-the-current-user-has-insufficient-permissions-to-pe)

### 영수증 검증 API

[go-iap](https://github.com/awa/go-iap)

```
import(
    "github.com/awa/go-iap/playstore"
)

func main() {
	// You need to prepare a public key for your Android app's in app billing
	// at https://console.developers.google.com.
	jsonKey, err := ioutil.ReadFile("jsonKey.json")
	if err != nil {
		log.Fatal(err)
	}

	client := playstore.New(jsonKey)
	ctx := context.Background()
	resp, err := client.VerifySubscription(ctx, "package", "subscriptionID", "purchaseToken")
}
```

#### 오류메시지

* 400, Invalid grant: account not found: 이 오류 메시지는 요청 인증 키가 잘못 생성되었음을 의미합니다. 계정이 연결되어 있고, 충분한 권한이 있는 올바른 계정을 사용하고 있으며, 필요한 모든 API가 활성화되어 있는지 확인하십시오. 모든 것을 구성하는 방법에 대한 가이드는 아래 섹션을 참조하세요. 제품 설명 업데이트에 대한 팁을 참고하세요.

* 400, The purchase token does not match the package name: 이 오류 메시지는 일반적으로 사기 거래에서 나타납니다. 테스트 중 보게 된다면, 다른 앱에 속한 앱 구매 토큰을 사용하고 있지는 않은지 확인하세요.

* 401, permissionDenied: The current user has insufficient permissions to perform the requested operation: 위에 json key 파일이 생성된지 24시간이 지나지 않아 아직 권한에 대해 미처리 상태입니다.

* ‍403, Quota exceeded for quota metric ‘Queries’ and limit ‘Queries per day’ of service ‘androidpublisher.googleapis.com’: 이것은 Google API 요청 일일 할당량이 초과되었음을 의미합니다. 기본적으로, 하루에 최대 200,000 요청을 실행할 수 있습니다. 할당량을 늘릴 수 있지만, 대부분의 앱에는 기본 할당량으로 충분합니다. 이 제한에 도달하면, 앱의 논리를 다시 점검하고 모든 것이 올바른지 확인해야 합니다.

* 410, The subscription purchase is no longer available for query because it has been expired for too long: 이 오류 메시지는 구독이 60일이나 그 전에 만료된 거래에 나타납니다. 실제 오류 메시지가 아니기 때문에 오류로 처리해서는 안 됩니다.

#### 파라미터 정의

* kind: 거래 유형. 구독의 경우, 항상 androidpublisher#subscriptionPurchase 값을 갖습니다. 이 매개변수를 통해, 구독을 처리하는지 또는 제품을 처리하는지 이해하고, 그에 맞는 처리 논리를 선택할 수 있습니다.

+ paymentState: 지불 상태. 만료된 거래에는 이 속성이 없습니다. 가능한 값은 다음과 같습니다.
	- 0: 이 구매는 아직 처리되지 않았습니다. 일부 국가에서는 사용자가 현장에서 구독료를 지불할 수 있습니다. 즉, 사용자는 자신의 장치에서 구독 구매를 시작하고 가까운 터미널에서 요금을 지불합니다. 전반적으로 매우 드문 경우이지만, 그래도 염두에 두어야 합니다. 
	- 1: 구독을 구매했습니다. 
	- 2: 구독은 평가판 (trial) 기간입니다. 
	- 3: 구독은 다음 기간에 업그레이드 또는 다운그레이드됩니다. 이는 구독 요금제가 변경되는 것을 의미합니다.

* acknowledgementState: 구매 확정 상태. 사용자가 지불한 것에 대한 접근 권한을 받았는지 여부를 승인하는 중요한 매개변수입니다. 값 ‘0’은 권한을 받지 않았음을, ‘1’은 받았음을 뜻합니다. 개발자는 이 상태를 정의할 책임이 있는데, 모바일 앱과 서버 측 모두에서 수행할 수 있습니다. 구매 시점으로부터 3일 이내에 승인하지 않으면, 자동으로 환불됩니다. 다음 논리를 구현하는 것이 좋습니다. acknowledgementState=0을 포함하는 거래를 받으면, 매개변수가 서버에 의해 변경됩니다. 그 방법은 아래에서 알려드리겠습니다.

* consumptionState: 제품 소비 상태. 이것이 iOS에서 ‘소모품’이라고 부르는 것입니다. 모바일 앱 측에서 정의됩니다. 값이 ‘0’이면 제품이 소비되지 않았음을 의미합니다. 만약 ‘1’이면 소비된 것입니다. 앱 또는 일부 특정 프리미엄 기능에 대한 평생 (lifetime) 접근 권한을 판매하는 경우, 이러한 제품은 소비되어서는 안 됩니다. 즉, 상태가 0이어야 합니다. 사용자가 몇 번이고 다시 살 수 있는 코인을 판매하는 경우, 해당 제품은 소비되어야 합니다. 즉, 상태가 1이어야 합니다. consumptionState=0은 제품을 한 번만 구입할 수 있음을 의미하지만, consumptionState=1은 여러 번 구매할 수 있음을 의미합니다.

* orderId: 고유한 거래 식별자. 각 구독 구매 또는 갱신에는 고유한 식별자가 있어서, 이 거래가 이미 이전에 처리되었는지 여부를 확인하는 데 사용할 수 있습니다.

* purchaseTimeMillis: 구입 일자.

* regionCode: 두 글자 형식으로 나타내는 구매 국가, (예: 미국). 이 매개변수의 이름은 구독에서는 countryCode라고 이름이 붙여져 있습니다.

* ‍purchaseType: 구매 유형. 이 키는 대부분의 경우에 존재하지 않습니다. 샌드박스 (Sandbox) 환경에서 구매했는지 여부를 이해하는 데 도움이 되기 때문에, 여전히 고려하는 것이 중요합니다. 가능한 값은 다음과 같습니다. 
	- 0: 샌드박스 환경에서 구매하였으므로, 분석 데이터에 포함되어서는 안 됩니다. 
	- 1: 프로모션 코드로 구매하였습니다.

* ‍cancelReason: 구독이 갱신되지 않는 이유. 가능한 값은 다음과 같습니다. 
	- 0: 사용자가 구독 자동 갱신을 취소했습니다. 
	- 1: 시스템에서 구독을 취소했습니다. 이것은 대부분 결제 문제로 인해 발생합니다. 
	- 2: 사용자가 다른 구독 요금제로 전환했습니다. 
	- 3: 개발자가 구독을 취소했습니다.	

[참고](https://adapty.io/blog/kr-android-in-app-purchases-part-5-server-side-purchase-validation)

## Voided Purchases API
## Reply to Reviews API