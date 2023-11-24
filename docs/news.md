## Flutter 3.13

- iOS 임펠러 엔진 성능개선(렌더링 최적화)
- Foldable API 추가
- AppLifecycleListener class 추가
- Scrolling
    - 가로/세로 양방향 스크롤가능한 layout
- Android 14/ API 34를 지원
- iOS 스크린 회전시 찌그러지는 현상 개선.
- Preparing for iOS 17 and Xcode 15
- Android Jelly Bean API levels (16, 17, and 18) 지원종료.

## Flutter 3.10

- iOS 임펠러 엔진이 기본값으로 적용.
- Web 환경 향상
- WASM 지원에 대해 향상
- iOS 무선 디버깅 지원
- Dart 3.0
    - 100% sound null safty
    - record, pattern
    - class 용도 구분 키워드 추가 (interface, base, final)

## Flutter 3.0.0

- Cross-Platform 지원
    - Flutter 3.0을 사용하면 개발자는 iOS, Android, 웹 및 데스크톱 등 다양한 플랫폼에서 사용할 수 있는 앱을 만들 수 있습니다. 이를 통해 이전보다 더 쉽게 모든 기기에서 앱을 사용할 수 있게 되어 사용자와의 폭넓은 도달 및 참여를 보장할 수 있습니다.
- Apple Silicon 지원
    - Intel 맥뿐만 아니라 M1 칩에 대해서도 지원하게 되었습니다.
- Foldable 디바이스 지원
- Desktop Window 지원
    - 이제 개발자는 창 UI(사용자 인터페이스)에 대한 향상된 지원 덕분에 모바일 장치뿐만 아니라 Mac OS X 또는 Windows PC와 같은 데스크톱 장치에서도 Flutter 3.0을 사용할 수 있습니다. 즉, 이제 모바일 장치에서 사용하는 것과 동일한 프레임워크를 사용하여 강력한 데스크톱 애플리케이션을 만들 수 있으므로 크로스 플랫폼 개발이 이전보다 훨씬 쉬워졌습니다!
- Web 기능 향상
    - Flutter Framework, Engine, Contents 로딩을 제어할 수 있는 새로운 API 추가
    - Image Decoding 성능 개선
- Material Design3 지원
- Dart 2.17
    - Enhanced enums with members
    - Super initializers
    - Named args everywhere
- 캐주얼 게임 툴킷 제공

## Go 1.20

- 1.19 이후 6개월 만의 릴리즈. 하위호환 정책으로 기존 프로그램 컴파일/실행은 문제 없음
- 언어에 4가지 변화
    - slice를 array로 변환 가능
    - unsafe 패키지에 SliceData, String, StringData 함수 추가
    - 구조체의 필드값이 정의에 나타난 순서대로 하나씩 비교되도록 하고, 첫번째 미스매치에서 중단되도록 정의됨. 비슷하게 배결 값도 하나씩 인덱스 순으로 비교
    - Comparable 타입들이 타입 인자가 strictly comparable 하지 않은 경우에도 comparable 조건을 충족 가능
- GC 데이터 구조 개선으로 메모리 오버헤드가 줄어들고 전체 CPU 성능 최대 2% 향상
- 그외 툴체인, 런타임, 라이브러리 구현등이 개선

## Swift 5.5

- type checker의 품질 및 성능 향상에 투자.

    → "expression too complex" 오류가 더 적게 표시됨

    → 배열 리터컬의 타입 검사 성능 향상

- 증분 빌드 속도를 높이기 위한 3가지 개선 사항
    - incremental imports 지원 == 모듈이 변경 될 때 모듈을 가져오는 모든 소스파일을 더이상 다시 빌드 하지 않음.
    - 모듈 종속성 그래프를 미리 계산하여 변경된 내용만 증분 빌드 == 빠르게 시작 가능
    - extensions과 함께 작동하도록 selective recompilation을 확장했음. == extension의 본문을 변경할 때 재 컴파일이 줄어든다.
- 메모리관리에 대한 효율 증가
    - ARC를 사용하여 객체에 대한 참조 수 추적.
- async await 추가
    - 비동기 처리가능.(콜백에 물리는 구조를 심플하게)    
- Actor
- Concurrency

## Go 1.19

- 메모리 모델을 C++/Java/Rust 등에서 사용하는 모델과 맞춤

    → Atomic 값을 사용하기 쉽게 atomic.Int64 및 atomic.Pointer[T] 같은 타입을 추가
    
- 가비지 수집기에 소프트 메모리 제한이 추가됐다. 이 제한을 통해 고 프로그램을 최적화해 메모리 양이 할당된 컨테이너에서 가능한 한 효율적으로 실행되도록 할 수 있다.
- 스택 카피라이팅을 줄이기 위한 코루틴 스택의 동적 크기 조정, 유닉스 시스템에서의 자동 추가 파일 설명자 사용, x86-64 및 Arm 64 상에서의 스위치 스테이트먼트용 점프 테이블, Arm64에서의 디버거 주입 함수 호출 지원이 지원된다. 
- 메소드 선언의 형식 매개변수가 수정됐다. 기존 프로그램은 영향을 받지 않는다.
- 이제 문서 주석에서 링크, 목록, 제목 구문을 지원한다. 특히 대규모 API를 갖춘 패키지에서 명확한 문서 주석을 작성할 수 있다. 
- 보안을 위해 os/exec 패키지는 PATH 룩업에서 더 이상 상대 경로를 고려하지 않는다. 
- 대상 OS가 유닉스 계열 운영 체제일 때 새로운 빌드 제약 조건인 unix가 충족된다. 


## Go 1.18

- Generic (제네릭) 추가
- Fuzzing (퍼징, 퍼즈 테스트) 추가
- Workspaces (work) 기능 추가
- 20% Performance Improvements


## TODO

- [x] go-redirect-server
<!-- - [ ] Test1
    * [x] Test2
    * [x] Test3
    * [ ] Test4
- [ ] Test5 -->
