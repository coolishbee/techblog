# 유니티의 메모리 관리

유니티 문서를 보면 언급되는 Managed Memory란 단서에서 흔히 추측하기로 유니티가 알아서 모든 메모리를 잘 관리하고 있을 것 처럼 생각되지만 사실은 그렇지 않다는 것이 함정.

사실은 유니티 시스템은 메모리를 어떻게 처리할 것인가에 대한 단서를 당신이 만드는 코드내에서 제공해주기를 원한다. 따라서 잘못된 코드는 줄줄 새는 메모리로 당신의 앱을 디바이스에서 뻗어 버리게 만들것이다. 요즘은 모바일 디바이스에서 조차 64bit 시스템(iPhone 5s)가 올라가고 기본 장착 메모리가 2Gb 이상이 되는 등 모바일 앱으로서는 무한한 자원이 있는것 같지만 사실상 아직도 지구상의 대부분의 유저는 허접한 디바이스로 연명하고 있다는 것을 기억해야 한다.

## 유니티 어플리케이션이 사용하는 메모리 종류

유니티에서 사용하는 메모리는 3가지 종류의 영역이 있다.

첫번째는 코드 영역으로 유니티 엔진과 라이브러리 그리고 당신이 만든 게임코드가 컴파일 되어서 디바이스의 메모리 코드 영역에 로딩되게 된다. 사실상 이 부분을 최적화 할 필요성은 크지 않다. 실제로 유니티 프로젝트에서 코드 텍스트 파일들이 차지하는 용량이 얼마나 되는지만 계산 해 봐도 알 수 있다. 유니티 코드가 얼마나 클지는 모르지만 예상컨데 큰 사이즈의 그림 파일 몇장에도 못 미치는 용량일것이다.

다음은 Managed Heap영역인데, 이 부분은 Mono가 관리하고 있는 영역이다. 아시다시피 Mono란 .Net Framework의 오픈 소스 버전이다. 이 Managed Heap영역에 클래스가 instantiated object, variable 들이 거주하게 되는 영역이다. "Managed"인 이유는 Mono Framework이 이 영역의 메모리를 할당하거나 해제하면서 "관리"를 하고 있기 때문이다. 실제로 어플리케이션이 실행중에 이따금씩 Mono가 garbage collection 작업을 통해 할당 되었으나 더이상 참조가 존재하지 않는 메모리 영역을 해제한다. 당신의 코드에서 더이상 사용되지 않는 변수나 클래스 오브젝트를 실수든 부주의에 의해서든 참조를 유지하고 있다면 해당 메모리 영역은 절대 해제되지 않고 메모리를 차지하고 있게 된다.

마지막으로 Native Heap 영역인데 이 부분은 Unity 엔진이 OS에서 메모리를 할당 받아 texture, sound effect, level data 등을 저장하고 있는 영역이다. 유니티 엔진이 이 영역을 담당하여 Mono가 메모리를 관리하는 것 처럼 현재 scene에서 필요 없게 된 리소스가 차지하는 메모리 영역을 해제하는 작업을 수행하기도 하나 manual하게 할당된 리소스에 대해서는 유니티가 관리를 하지 못하므로 부주의 하다면 심각한 메모리 부족 사태에 이를 수 있다.

## 어플리케이션 코드 영역 메모리 최적화

어플리케이션 코드 영역을 최적화하는 작업은 비교적 쉽게 프로젝트 세팅을 바꾸어 주는 것 만으로도 해결할 수 있다. 그러나 이 최적화 옵션을 바꿈으로 인해 사용가능하지 않을 feature를 당신의 어플리케이션이 사용하지 않을 것 인지 확인할 필요가 있다.

사실 Mono Framework는 수많은 .Net 라이브러리를 포함하고 있는데 이 모든 라이브러리를 로딩하는데 꽤 많은 메모리 리소스를 소모하게 된다. 유니티에서는 이 overhead를 줄일 수 있는 옵션을 제공하는데 유니티 메뉴중 Edit > Project Settings > Player 메뉴를 선택하여 보자.

![](http://3.bp.blogspot.com/-bE_kvYsfQ3U/UkJFSed9MSI/AAAAAAAAEBI/x8pvxRMTD9U/s400/unity_option.jpg)

위와 같은 옵션을 볼 수 있을 것인데 중요한것은 아래쪽 Optimization 부분의 옵션들이다. "Api Compatibility Level"옵션을 ".NET 2.0 Subset"으로 바꾸어 유니티에게 더 적은 수의 .Net 라이브러리를 사용하도록 알릴 수 있다. 그리고 그 아래 "Stripping Level"의 옵션을 바꾸어 더 적은 용량의 빌드된 코드를 사용하도록 할 수 있다.  (Stripping Level 옵션의 자세한 내용에 대해서는 [링크](http://docs.unity3d.com/Documentation/Manual/iphone-playerSizeOptimization.html)를 참조하도록 하자.) 해당 옵션의 수정을 통해 빌드 사이즈를 줄일 수 있으며 줄어든 빌드 사이즈는 곧 더 적은 메모리 사용을 의미한다. Stripping Level옵션은 Unity Pro에서만 제공되는 기능이다.

그럼 그냥 가장 minimal한 것으로 설정을 하면 되겠지 싶은데 세상일이 그렇게 간단하지 않다. 그 전에 당신의 어플리케이션이 옵션 조정으로 인해 빠지게 되는 라이브러리를 사용하지 않는지 확인할 필요가 있다. 가장 많이 쓰지만 이외로 minimize되서 빠지게 되는 라이브러리 중에는 System.Xml이 있다. 이 경우는 3rd party minimal 라이브러리를 사용하면 된다.

옵션에 따라 지원되는 라이브러리를 확인할 수 있는 페이지는 [이곳](http://docs.unity3d.com/410/Documentation/ScriptReference/MonoCompatibility.html)을 참조하도록 하자. 실제로 빠지게 되는 많은 라이브러리들이 게임을 개발할 때는 잘 사용되지 않는 것들이 많다. 따라서 한번 시도해 볼만한 가치가 충분히 있다고 본다.

## Managed Heap 영역의 메모리 최적화

유니티에서 최근에 Managed Heap 영역의 메모리를 관리하는 법에 대해 훌륭한 [메뉴얼](http://docs.unity3d.com/Documentation/Manual/UnderstandingAutomaticMemoryManagement.html)을 추가하였다. Managed Heap영역은 당신이 만든 코드에서 new 키워드나 Instantiate() 함수를 통해 할당한 메모리가 Mono Framework에 의해 관리되는 영역이다. 만약 메모리가 필요해 할당을 하는데 Manges Heap영역에 메모리가 모자랄 경우 heap 사이즈를 키우게 된다. 이 경우가 실제로 디바이스 상에서 메모리가 부족해지는 경우이다.

일반적으로는 어떤 인스턴스가 더이상 필요해지지 않을때 해당 인스턴스를 참조하는 곳이 없어지게 된다. 주기적으로 Mono의 Garbage Collector가 이러한 참조가 없어진 인스턴스 등을 찾아 Manged Heap상에서 해제하게 된다. 그러나 Garbage Collector가 메모리를 수거하는 작업을 하는 동안에는 framerate가 떨어지거나 어플리케이션의 반응이 늦어질 수가 있기 때문에 일반적으로 어플리케이션이 활발히 동작하고 있는 동안에 Garbage Collector의 동작을 원치는 않을 것이다. 따라서 메모리가 새는 것을 방지하기 위해 할당한 변수나 오브젝트가 필요가 없어지게 되면 null을 할당하거나 Destory로 명시적으로 메모리상에서 제거되도록 표시를 해두어야 하지만 동시에 수많은 메모리 해제로 과도한 Garbage Collector의 동작을 바라지도 않을 것이다.

특히 게임의 경우 Destory를 호출해야만 하는 수많은 object가 게임 플레이 도중에 생성될 수 있다. 예를 들면 대포가 수많은 대포알을 발사하는 경우가 그렇다. 각 대포알을 발포 되기전에 메모리를 할당 받고 대포알이 사라지는 순간 해제해야 하는데 이러한 작업이 각 대포알에게 일어나야 하는 것은 너무나도 낭비적인 일이 아닐 수 없다. 이 문제를 해결하는 일반적인 방법은 해당 오브젝트를 생성후 필요 없게 되는 때에 Destory하지 않고 일단 숨긴 후 필요할 때 다시 Activate시켜서 사용하는 이른바 Object Recycling 기법이 있다. 항상 이 방법이 유용한것은 아니지만 과도한 Garbage Collection을 피하는 훌륭한 방법이다. 

Automatic Memory Management와 Object Pool에 관련해서는 [이곳](http://docs.unity3d.com/Documentation/Manual/UnderstandingAutomaticMemoryManagement.html)의 유니티 문서를 통해 자세한 사항을 확인하도록 하자.

Garbage Collection이 일어나는 것을 막거나 컨트롤 할 수 있는 방법은 딱히 없지만 시스템에 적당한 시점에 지금이 바로 Garbage Collection을 하기에 좋은 타이밍이라는 것을 힌트를 줄 수는 있다. System.GC.Collect()를 호출하는 방법이 그것인데 이 함수를 호출하는 것이 바로 Garbage Collection을 하도록 하지는 않고 정말 단지 지금이 좋은 타이밍이라는 것을 알려줄 수 있을 뿐이지만 적당한 시점에 예를 들어 화면이 전환되는 시점이나 리소스 로딩이 일어나는 시점에 호출해준다면 큰 효과를 볼 수 있다. 

## Native Heap 영역의 메모리 최적화

Unity에서 Scene을 로드하면 그 Scene 에 포함된 모든 Asset들을 같이 메모리 상에 로드되고 이어서 해당 Scene이 유저에게 보여진다. 그리고 해당 Scene이 끝나서 다음 Scene으로 넘어갈 경우 이전 Scene의 리소스중 다음 Scene에서도 사용되지 않는 다면 Asset은 메모리상에서 자동으로 해제된다.

그러나 어떤 두가지의 경우에 Asset이 자동으로 해제되지 않는 경우가 있는데, 첫번째로는 DontDestroyOnLoad(Object target) 함수를 통해 개발자에 의해 다른 Scene이 Loading 되더라도 Destroy되지 않도록 지정해둔 경우로 이 경우 해당 GameObject와 GameObject의 Child GameObject, 연계된 Asset 모두가 Scene 전환에 따라 자동 해제되지 않는다. 실제로 이 함수는 모든 Scene간에 보존되어야 하는 데이터를 전달하는 방법으로 사용되고 있는데 이 경우에 보존되는 Object가 무거운 Asset을 참조하거나 그 자체가 아니도록 주의하여야 한다.

또 다른 경우는 Script내에서 Scene Object를 참조하고 있는 경우이다. 대부분 Script의 참조는 Scene이 전환됨에 따라 같이 파괴되게 되지만 Scene이 전환되더라도 남아 있는 GameObject가 있고 해당 GameObject 가 Script를 가지고 있는데 이 Script내에서 다른 Asset 예를 들어 사운드 이펙트나 텍스쳐에 대한 참조를 유지하고 있는 경우 무심코 필요하지 않은 리소스가 메모리상에 남아서 메모리 자원부족에 이르게 할 위험이 있다. 또 Static 변수 혹은 Singleton인스턴스가 자원에 대한 참조를 가지고 있는 경우도 마찬가지 문제가 발생할 소지가 있다.

따라서 스크립트내에서 이러한 참조를 유지하고 있다면 필요없게 되는 시점에 잊지않고 Destroy함수를 호출하거나 null을 할당하여 참조를 해제할 필요가 있다. 

유니티는 Scene이 Load되는 시점에 해당 Scene의 모든 Asset을 자동으로 같이 Load하지만 이러한 Asset을 해제할 유일한 방법은 Scene을 다시 로드하거나 다른 Scene으로 전환하는 방법밖에 없다는 것에 주의할 필요가 있다.

## 리소스 로딩을 수동으로 관리하기

약간의 노동이 필요하지만 수동으로 Asset을 현재 Scene에 Load할 방법도 당연히 존재한다. Resources 라는 폴더를 만들어서 Asset들을 그 안에 위치시키면 해당 Asset들은 Resources.Load(resourcePath)함수를 통해 수동으로 Load할 수 있게 된다.  이 Asset들은 텍스쳐, 오디오파일, 프리팹, 매터리얼등이다. 

해당 자원에 대한 이용이 끝났다면 그 자원에 할당된 메모리를 Resources.UnloadUnusedAssets() 함수를 호출하여 강제로 해제할 수 있는데, 유니티가 해당 자원을 해제하기 위해서는 스크립트에서 해당 자원에 대한 참조가 모두 없어져야 한다. (Destroy혹은 null을 할당해서) 그리고 이 함수는 Resources.Load() 함수를 통해서 할당된 자원만 해제를 하고 유니티가 Scene을 Load하면서 자동으로 같이 Load한 자원은 해당되지 않는다는 점에 주의할 필요가 있다.

!!! Reference

    [http://la-stranger.blogspot.kr](http://la-stranger.blogspot.kr)