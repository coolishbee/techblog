## Overview

보안 및 실행 효율성을 개선하기 위해, 시장의 대부분의 유니티 게임은 C# 기반 Mono 대신 네이티브 코드 기반 IL2CPP를 선택합니다. 네이티브 코드가 크래킹을 더 어렵게 만들지만, 치트 개발자는 여전히 Il2CppDumper와 게임의 global-metadata.dat 파일을 사용하여 libil2cpp.so의 모든 기호 및 문자열 정보를 복원할 수 있습니다. "Il2cpp"와 "cracking"과 같은 키워드를 구글링하면 많은 사용법을 찾을 수 있습니다.

그래서 유니티엔진으로 클라이언트를 개발하면서 치트를 방지하는 전략은 다음과 같습니다.

* Resource, Data 보호 -> 바이너리로 암호화
* 소스코드 보호(Android Proguard, R8)
* 변수 메모리 난독화
* global-metadata.dat 보호

보통 클라이언트에서 완벽하게 Anti-Cheat 할 수 없지만 이렇게 해커들이 치트하기 어렵고 불편하게 만들 수 있습니다.
이 중에서 유니티 IL2CPP 컴파일러의 특성때문에 존재하는 global-metadata.dat 보호에 대해 간략하게 설명드리려고 합니다.

## global-metadata.dat 은 무엇인가?

IL2CPP(AOT) 컴파일 방식에서는 Literal String, Method, Struct 정보들을 `global-metadata.dat`라는 파일에 저장하고 보관합니다. 게임이 실행될 때 이 파일을 직접 메모리에 로드하고 파싱하여 배열로 관리됩니다.

```cpp
bool il2cpp::vm::GlobalMetadata::Initialize(int32_t* imagesCount, int32_t* assembliesCount)
{
    s_GlobalMetadata = vm::MetadataLoader::LoadMetadataFile("global-metadata.dat");
    if (!s_GlobalMetadata)
        return false;

    s_GlobalMetadataHeader = (const Il2CppGlobalMetadataHeader*)s_GlobalMetadata;
    IL2CPP_ASSERT(s_GlobalMetadataHeader->sanity == 0xFAB11BAF);
    ...

    return true;
}
```

실제 유니티엔진에서 초기화하는 소스코드입니다. 이렇게 global-metadata.dat 문자열 그대로 사용하기 때문에 엔진소스코드가 난독화되어도 문자열은 난독화가 되지 않습니다.

그렇기 때문에 [IL2CPPDumper](https://github.com/Perfare/Il2CppDumper)를 이용해서 게임정보를 전부 들여다 볼 수 있으며 메모리 변조도 아주 쉽게 가능한 것입니다. 그리고 시도해보진 않았지만 `global-metadata.dat`파일명을 변조한다고 해도 큰 효과를 보기 어려워보이는 것은 `0xFAB11BAF`라는 해당 파일의 매직넘버가 고정적이기 때문에 런타임 메모리덤프를 통해서 찾아내기 쉬울거라고 생각합니다.

그래서 `global-metadata.dat`이 추출이 되면 왜 위험하고 어떻게 Cheat를 하는 건지에 대해 답한다면 클래스명, 변수명, 자료형, 코드 로직까지 모두 재구성이 가능해지기 때문입니다.

## global-metadata.dat 보호에 대한 단계별 전략

다음과 같이 모든 단계는 유니티의 OnPostprocessBuild 통하여 변환할 생각입니다.

유니티 빌드시 `global-metadata.dat`의 경로는 고정적입니다. 이를 이용해서 다양한 시도를 해볼 수 있을 거 같습니다.

```csharp
private const string METADATA = "Library/Bee/Android/Prj/IL2CPP/Gradle/unityLibrary/src/main/assets/bin/Data/Managed/Metadata/global-metadata.dat";

class MyCustomBuildProcessor : IPostprocessBuildWithReport
{
    public int callbackOrder { get { return 0; } }
    public void OnPostprocessBuild(BuildReport report)
    {
        //File io
    }
}
```

### 1단계

가장 빠르게 시도해 볼 수 있는 방법입니다. 

`global-metadata.dat`파일명을 매번 빌드할때마다 랜덤한 파일명을 만들어내어 [IL2CPPDumper](https://github.com/Perfare/Il2CppDumper)가 찾아내지 못하게 하는 방법입니다.

PostBuild 때 파일명을 바꾸고 `GlobalMetadata.cpp`에 접근하여 로드되는 파일명을 수정해주면 됩니다.

*물론 동적메모리 덤프시 뚫릴는 수는 있으나 당장 공급되는 해킹툴의 작동은 멈출 수 있는지 정도만 확인해보아도 유미의하다 생각됩니다.*

### 2단계

이번엔 `global-metadata.dat`파일을 아에 숨기는 방법입니다.

파일형태가 아닌 파일을 로드해 임의의 `byte array`로 변환하여 `header`파일에 쓰여 저장되게 합니다.
즉, `GlobalMetadata.cpp`에서 파일형태가 아닌 바이너리를 직접 메모리카피 해주고 리턴해줍니다.

여기까지도 공격자가 어느정도 노력한다면 뚫어내기 어렵지 않을 것 같습니다.

### 3단계

위 2단계에서 변환한 `byte array`를 `xor`로 암복호화 한다면 아주 효과적인 방어가 될 것 같습니다.

아마도 이렇게 한다면 지금처럼 쉽게 메모리 변조가 일어나지는 않을 거라고 예상합니다.


## 결론

결국 위에 언급한 4가지 Anti-Cheat 전략을 다 포함하는 것이 가장 최선의 시너지를 낼 것이며
`global-metadata.dat`만 숨기거나 어떤 한가지의 보호만 이루어진다면 공격자로 부터 더 쉬운 Cheat를 제공하는 것이라고 생각합니다.