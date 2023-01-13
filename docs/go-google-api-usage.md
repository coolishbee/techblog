# Google API 사용법

go 언어용 라이브러리가 있지만 직접 액서스 토큰으로 Google API 를 사용하는 방식입니다.

## API 콘솔 프로젝트 생성 및 설정

[Google Play Developer API](https://developers.google.com/android-publisher/authorization?hl=ko) 을 사용하기 위해선 OAuth2 인증이 필수입니다.
Google Cloud 프로젝트내 사용자 인증 정보에서 웹 애플리케이션 유형으로 OAuth 2.0 클라이언트 ID 를 만들어줘야 합니다.
(참고로 토큰을 얻기 위해 OAuth 동의화면에서 아무런 API 범위 추가가 필요하지 않습니다.)

토큰을 얻기 위해 필요한 필수 항목

* ClientID
* ClientSecret
* 승인된 리디렉션 URI

## RefreshToken 생성

### Redirect 서버 구축

토큰을 얻기위해 미리 Redirect 서버를 구축해야 합니다.
 
[example](https://github.com/coolishbee/go-redirect-server) :
```go
package main

import (
	"context"
	"log"
	"net/http"

	"github.com/gin-gonic/gin"
	"golang.org/x/oauth2"
	"golang.org/x/oauth2/google"
)

func main() {
	router := gin.Default()
	ctx := context.Background()

	oauth2Conf := &oauth2.Config{
		ClientID:     "871xxxxxxxx-xxxxx.apps.googleusercontent.com",
		ClientSecret: "GOXXXX-31xxxxxxxx",
		Endpoint:     google.Endpoint,
		RedirectURL:  "http://localhost:8077/redirect",
		Scopes: []string{
			"https://www.googleapis.com/auth/admob.readonly",
		},
	}

	router.GET("/redirect", func(c *gin.Context) {
		code := c.Query("code")
		log.Println(code)
		tok, err := oauth2Conf.Exchange(ctx, code)
		if err != nil {
			c.JSON(500, err)
			return
		}
		log.Println(tok.RefreshToken)
	})
}  
```

### 브라우저에서 로그인하기

미리 구글계정을 로그아웃해놓고 아래 URI 로 요청 후 로그인 합니다.
로그인이 성공한다면 리디렉션 서버로 요청이 갑니다. 그리고 code를 받아서 exchange 하면 RefreshToken 이 발급됩니다.

* scope : 사용할 API 범위
* client_id : API 콘솔에서 설정한 ClientID

```
https://accounts.google.com/o/oauth2/auth?scope=https://www.googleapis.com/auth/admob.readonly&response_type=code&access_type=offline&redirect_uri=http://localhost:8077/redirect&client_id=...
```

## API 사용하기

백엔드단에서 매번 브라우저로 인증처리를 할 수 없기때문에 RefreshToken 을 이용해서 AccessToken 을 얻습니다.

### AccessToken 얻기위한 POST 요청

```
POST
https://accounts.google.com/o/oauth2/token
HEADER
"Content-Type" : "application/x-www-form-urlencoded"
```
[Request Body (FormData)]
| Key  | Description     | Type   |
| ---- | --------------- | ------ |
| refresh_token   | 위에서 발급받은 토큰   | string |
| client_id | API 콘솔에서 얻은 id     | string |
| client_secret  | API 콘솔 key | string |
| grant_type  | 어떤 유형으로 토큰을 얻을지에 대한 타입 | string |

example :
```go
import "github.com/go-resty/resty/v2"

func example() {
	client := resty.New()
	resp, err := client.R().
		SetHeader("Content-Type", "application/x-www-form-urlencoded").
		SetFormData(map[string]string{
			"refresh_token": "1//0e6caG1CsCnrVCgYIARAAGA4SNwF-...",
			"client_id":     "871390000-xxx.apps.googleusercontent.com",
			"client_secret": "GOXXXX-31xxxx",
			"grant_type":    "refresh_token",
		}).
		Post("https://accounts.google.com/o/oauth2/token")

	if err != nil {
		log.Println(err.Error())
	}	
	fmt.Println("  Body       :\n", resp)
  ...
```

### Google Developer API 호출

admob 예제:
```go
type AdmobReq struct {
	ReportSpec struct {
		DateRange struct {
			StartDate struct {
				Year  int `json:"year"`
				Month int `json:"month"`
				Day   int `json:"day"`
			} `json:"startDate"`
			EndDate struct {
				Year  int `json:"year"`
				Month int `json:"month"`
				Day   int `json:"day"`
			} `json:"endDate"`
		} `json:"dateRange"`
		Metrics              []string `json:"metrics"`
		LocalizationSettings struct {
			CurrencyCode string `json:"currencyCode"`
			LanguageCode string `json:"languageCode"`
		} `json:"localizationSettings"`
		Dimensions []string `json:"dimensions"`
	} `json:"reportSpec"`
}

var admobReq AdmobReq
admobReq.ReportSpec.DateRange.StartDate.Year = startYear
admobReq.ReportSpec.DateRange.StartDate.Month = startMonth
admobReq.ReportSpec.DateRange.StartDate.Day = startDay
admobReq.ReportSpec.DateRange.EndDate.Year = endYear
admobReq.ReportSpec.DateRange.EndDate.Month = endMonth
admobReq.ReportSpec.DateRange.EndDate.Day = endDay
admobReq.ReportSpec.Metrics = []string{"ESTIMATED_EARNINGS", "IMPRESSIONS", "IMPRESSION_RPM"}
admobReq.ReportSpec.LocalizationSettings.CurrencyCode = "USD"
admobReq.ReportSpec.LocalizationSettings.LanguageCode = "en-US"
admobReq.ReportSpec.Dimensions = []string{"DATE", "APP"}

reportResp, err := client.R().
  SetAuthToken(authResp.AccessToken). //마샬링한 액서스 코드
  SetHeader("ContentType", "application/json").
  SetBody(admobReq).
  Post("https://admob.googleapis.com/v1/accounts/pub-0000000/networkReport:generate")

fmt.Println("  Error      :", err)
fmt.Println("  Body       :\n", reportResp)
```

!!! tip

    동일한 방식으로 구글 영수증검증, 환불목록 조회 API 등 모든 API 사용이 가능합니다.