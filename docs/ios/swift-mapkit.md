---
comments: true
---

## CoreLocation

https://co-dong.tistory.com/73
https://ios-daniel-yang.tistory.com/77

### desiredAccuracy

앱이 받고자 하는 위치 데이터의 정확성.

```swift
var desiredAccuracy: CLLocationAccuracy { get set }
```

사용자의 위치를 활용하는 컨텐츠 중요도에 따라서 선택하면 좋을 듯 하다.
높은 정확도를 이용할수록 배터리 소모라는 리스크가 뒤 따른다.

- kCLLocationAccuracyBestForNavigation : 내비게이션 앱을 용이하게 하기 위해 추가 센서 데이터를 사용하는 가능한 가장 높은 정확도.
- kCLLocationAccuracyBest : 이용 가능한 최고 수준의 정확도.
- kCLLocationAccuracyNearestTenMeters : 원하는 목표에서 10미터 이내로 정확하다.
- kCLLocationAccuracyHundredMeters : 100미터 이내로 정확하다.
- kCLLocationAccuracyKilometer : 가장 가까운 킬로미터까지 정확하다.
- kCLLocationAccuracyThreeKilometers : 가장 가까운 3킬로미터까지 정확하다.