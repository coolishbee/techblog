## 배포 자동화 구축 목적

혼자서 많은 작업을 하다보니 배포시간으로 낭비되는 시간을 줄이기 위해 자동화를 선택했습니다.

### 나에게 주어진 역할

- Android SDK

- iOS SDK

- Unity SDK

- 유니티 Android 데모앱

- 유니티 iOS 데모앱

- [SDK Guide Docs](https://docs.igamepub.co.kr) 구축 및 관리



## 젠킨스 구축

SDK 배포와 SDK 테스트용 데모앱 배포를 위해 [Fastlane](https://fastlane.tools) 과 [Jenkins](https://www.jenkins.io) 를 사용했습니다.

- 젠킨스 설치

- 블루오션 플러그인 설치

- 유니티 플러그인 설치



## iOS SDK 빌드

iOS SDK 경우에는 배포사이클이 길다보니 [Pipeline](https://www.jenkins.io/doc/book/pipeline/syntax) 을 사용하는게 직관적이고 관리하기 용이할 것 같아서 스크립트로 관리했습니다.

그리고 나중에 혹시 병렬로 처리했을때 확인이 어려울 수 있으니 [Blue Ocean](https://www.jenkins.io/projects/blueocean) 을 설치하여 빌드 과정을 시각화했습니다.

- pod install

- fastlane build

- Deploy 



![](https://user-images.githubusercontent.com/20632507/148344197-acd533cd-1c06-492a-8043-450cc4439ef9.png)



## 유니티 Android 빌드

네이티브 SDK 배포 횟수만큼이나 유니티 빌드도 굉장히 많이 하기 때문에 구성하게 되었습니다. 

프로젝트가 유니티 SDK 이다보니 주로 유니티 데모앱을 통해서 기능 테스트가 이루어지는데 아주 빈도수가 높습니다.

또한 apk 를 빌드후 테스트폰에 설치하는 행위도 고려하여 APK 다운로드페이지를 구축했습니다.

- 젠킨스내 유니티 플러그인 설치 및 설정

- 유니티 cli 실행시 파라미터 설정

- shell 스크립트 작성

- apk 업로드



![](https://user-images.githubusercontent.com/20632507/148345487-79015afc-4bfc-4348-9230-73fb43a16711.png)

![](https://user-images.githubusercontent.com/20632507/148345509-eda7f14d-85de-42f5-9e7b-df1e437586d2.png)

![](https://user-images.githubusercontent.com/20632507/148345858-8a08d16d-cba6-42b6-bed2-5e07570d316d.png)


