
## Intent

Component를 실행하게 해주는 역할.<br>
실행하고자 하는 컴포넌트 정보를 담은 Intent구성 -> 시스템 -> Intent 정보를 통해 그에 맞는 Component를 실행하게 된다. 그리고
안드로이드는 Component기반의 구조이고 이때 Intent에 의해 내부적으로 개발자가 작성한 Activity같은 클래스들이 아래와 같이 동작하게 된다.

안드로이드 어플리케이션을 구성하는 네 가지 기본 요소에는 Activity, Service, Broadcast Receiver, Content Provider가 있다.
이때 인텐트(Intent)란 이러한 어플리케이션 구성요소(컴포넌트) 간에 작업 수행을 위한 정보를 전달하는 역할을 한다.

즉, 인텐트는 컴포넌트 A가 컴포넌트 B를 호출할 때 필요한 정보를 담고 있으며, 이 정보에는 호출되는 컴포넌트 B의 이름이 명시적으로 표시거나, 속성들이 암시적으로 표시되기도 한다.

## 안드로이드를 구성하는 Component

Activity

액티비티는 UI를 담당하는 컴포넌트입니다. 즉, 사용자에게 보여지는 화면입니다. 어플리케이션은 반드시 1개 이상의 액티비티를 가지고 있어야 합니다.

Service

서비스는 백그라운드에서 실행되는 컴포넌트입니다. 오랫동안 실행되는 작업이나 원격 프로세스를 위한 작업을 수행합니다. 메인 쓰레드에서 실행되므로 서비스에서 별도의 스레드를 생성해서 작업을 처리해야 합니다.

BroadCast Receiver

브로드캐스트 리시버는 안드로이드 OS로부터 발생하는 이벤트와 정보(BroadCast) 를 핸들링하는 컴포넌트입니다. 브로드캐스트에는 시스템 부팅 완료, 배터리 부족, 언어설정 변경, 문자메세지 수신 등이 있습니다. 10초 이내의 작업만 보증하므로 오랜 시간 동작해야한다면 별도의 쓰레드나 서비스로 처리해야 합니다.
Broadcast Receiver 를 등록하는 방법에는 2가지가 있습니다. AndroidManifest 에 등록하는 정적 등록과 코드상에서 등록하는 동적 등록이 있습니다.

Content Provider

컨텐트 프로바이더는 애플리케이션 사이에서 각 데이터들을 공유할 수 있도록 하는 구성요소입니다. 안드로이드는 기본적으로 주소록, 이미지, 오디오 등 주요 데이터에 대한 내장 Content Provider를 제공합니다.

## 액티비티 생명주기

![activity-lifecycle](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-activity-lifecycle/img/468988518c270b38_856.png?hl=ko)

## 프래그먼트 생성주기

![fragment-lifecycle](https://developer.android.com/static/images/guide/fragments/fragment-view-lifecycle.png?hl=ko)