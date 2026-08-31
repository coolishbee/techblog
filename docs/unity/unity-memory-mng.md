---
comments: true
---

# Unity 6 메모리 관리와 프로파일링

## 들어가며

Unity의 메모리 문제는 단순히 관리되는 힙이 커지는 현상만을 뜻하지 않는다. C# 객체, Unity 네이티브 객체, 텍스처와 메시 같은 그래픽 리소스, 운영체제 메모리가 함께 움직인다.

따라서 최적화의 시작점은 설정 변경이나 강제 GC가 아니라 측정이다. 어떤 메모리가 언제 증가하고, 참조가 남아 있는지, 프레임 스파이크가 GC와 관련 있는지를 먼저 분류해야 한다.

!!! warning "측정 없이 최적화하지 않는다"
    앱 크기 축소, 관리되는 할당 감소, 네이티브 리소스 해제는 서로 다른 문제다. Profiler와 Memory Profiler로 원인을 확인한 뒤 가장 작은 변경부터 적용한다.

## Unity 애플리케이션의 메모리 모델

| 영역 | 주로 포함하는 항목 | 흔한 문제 | 확인 방법 |
| --- | --- | --- | --- |
| 관리되는 메모리 | C# 객체, 컬렉션, 문자열, 클로저 | 프레임마다 발생하는 할당, 오래 유지되는 참조 | Profiler의 GC.Alloc, 관리 힙 지표 |
| Unity 네이티브 메모리 | GameObject, Component, AnimationClip, AudioClip 등의 엔진 측 데이터 | 파괴되지 않은 객체, 씬 전환 뒤 남는 참조 | Memory Profiler 스냅샷 |
| 그래픽 메모리 | 텍스처, 메시, 렌더 타깃, 셰이더 리소스 | 고해상도 에셋, 중복 로드, 릴리스 누락 | Memory Profiler, 플랫폼 도구 |
| 시스템 메모리 | Unity 할당자, 네이티브 플러그인, 운영체제 메모리 | 플러그인 누수, 메모리 압박, 플랫폼별 한도 | System Used Memory, 기기 도구 |

UnityEngine.Object는 C# 래퍼와 엔진의 네이티브 데이터를 함께 가진다. C# 참조를 잃었다고 해서 모든 네이티브 리소스가 즉시 사라진다고 가정하면 안 된다. 반대로 관리되는 객체가 남아 있으면 연결된 에셋과 네이티브 객체가 예상보다 오래 유지될 수 있다.

## 먼저 원인을 분류한다

메모리 증가에는 대체로 두 가지 패턴이 있다.

=== "일시적 할당이 많은 경우"

    매 프레임 또는 자주 실행되는 경로에서 문자열 결합, LINQ, 컬렉션 생성, Instantiate 등이 반복된다. GC.Alloc과 프레임 시간에 스파이크가 함께 나타나는지 확인한다.

=== "참조가 오래 남는 경우"

    씬을 전환해도 싱글턴, static 필드, 이벤트 구독, DontDestroyOnLoad 객체, 에셋 로더의 핸들이 리소스를 계속 가리킨다. 전후 스냅샷을 비교해 살아 있는 참조 경로를 찾는다.

!!! tip "증상과 원인을 분리"
    메모리가 증가한다고 항상 누수는 아니다. 캐시, 풀, Unity 할당자 예약 메모리처럼 의도된 증가일 수 있다. 문제는 목표 시나리오가 끝난 뒤에도 메모리와 참조 수가 계속 증가하는지다.

## 권장 진단 절차

1. 실제 문제가 발생하는 대상 기기와 동일한 품질·에셋·빌드 설정으로 재현한다.
2. Profiler에서 프레임 시간, GC.Alloc, Managed Heap, System Used Memory의 변화를 확인한다.
3. 문제 시나리오 전과 후에 Memory Profiler 스냅샷을 만든다.
4. 두 스냅샷의 객체 수, 크기, 참조 경로를 비교해 관리·네이티브·그래픽 메모리 중 증가한 영역을 분류한다.
5. 한 가지 원인만 수정한 뒤 같은 시나리오를 다시 실행해 수치와 프레임 시간을 비교한다.

!!! note "스냅샷은 비교해서 사용한다"
    단일 스냅샷은 현재 상태만 보여 준다. 씬 진입 전, 반복 실행 후, 씬 이탈 후처럼 같은 기준점의 두 스냅샷을 비교해야 의도치 않게 남는 객체를 찾기 쉽다.

## 런타임 지표를 기록하는 예제

ProfilerRecorder는 Profiler가 노출하는 메모리·성능 카운터를 코드에서 읽을 수 있게 한다. 아래 예제는 진단용 컴포넌트로 GC 예약 메모리와 시스템 사용 메모리를 주기적으로 출력한다.

~~~csharp
using Unity.Profiling;
using UnityEngine;

public sealed class MemoryMetricLogger : MonoBehaviour
{
    private const int LogIntervalFrames = 300;

    private ProfilerRecorder _gcReservedMemory;
    private ProfilerRecorder _systemUsedMemory;

    private void OnEnable()
    {
        _gcReservedMemory = ProfilerRecorder.StartNew(
            ProfilerCategory.Memory,
            "GC Reserved Memory");
        _systemUsedMemory = ProfilerRecorder.StartNew(
            ProfilerCategory.Memory,
            "System Used Memory");
    }

    private void Update()
    {
        if (Time.frameCount % LogIntervalFrames != 0)
        {
            return;
        }

        Debug.Log(
            "GC Reserved: " + ToMegabytes(_gcReservedMemory.LastValue)
            + " MB, System Used: "
            + ToMegabytes(_systemUsedMemory.LastValue)
            + " MB");
    }

    private void OnDisable()
    {
        _gcReservedMemory.Dispose();
        _systemUsedMemory.Dispose();
    }

    private static double ToMegabytes(long bytes)
    {
        return bytes / (1024d * 1024d);
    }
}
~~~

이 코드는 경향을 확인하는 도구일 뿐, 누수 여부를 단독으로 판정하지는 않는다. 진단이 끝나면 제거하거나 개발용 빌드에서만 활성화한다. ProfilerRecorder는 관리되지 않는 리소스를 사용하므로 더 이상 필요하지 않을 때 Dispose로 해제해야 한다.

## 관리되는 할당을 줄이는 방법

GC 스파이크를 줄이는 가장 안전한 방법은 자주 실행되는 경로의 불필요한 할당을 없애는 것이다.

| 패턴 | 점검 방법 | 개선 방향 |
| --- | --- | --- |
| 매 프레임 문자열 생성 | Update, UI 갱신, 로그 경로의 GC.Alloc 확인 | 변경이 있을 때만 문자열을 만들고 버퍼를 재사용한다 |
| 반복 컬렉션 생성 | LINQ, 배열·List 생성 경로 확인 | 컬렉션 용량을 미리 확보하고 재사용한다 |
| Instantiate와 Destroy 반복 | 투사체, 이펙트, UI 생성 빈도 확인 | 짧은 수명의 동일 객체에 한해 풀링을 검토한다 |
| 컴포넌트 반복 탐색 | GetComponent 계열 호출 위치 확인 | 필요한 참조를 초기화 단계에 캐시한다 |
| 이벤트 구독 해제 누락 | 싱글턴과 장수 객체의 구독 관계 확인 | 수명 종료 시점에 구독을 해제한다 |

!!! warning "풀링은 측정된 병목에만 적용"
    풀은 생성·파괴 비용과 할당을 줄일 수 있지만, 너무 큰 풀은 메모리를 계속 점유한다. 자주 재사용되는 동일한 객체라는 근거가 있을 때만 크기 제한과 반납 시점까지 설계한다.

## 리소스 수명 관리

씬 전환만으로 모든 리소스가 해제된다고 가정하지 않는다. 특히 다음 항목은 스냅샷에서 우선 확인한다.

- DontDestroyOnLoad 객체가 무거운 에셋 또는 씬 객체를 참조하는지
- static 필드, 싱글턴, 캐시가 이전 씬의 객체를 보관하는지
- 이벤트·콜백·비동기 작업이 종료된 객체를 계속 참조하는지
- 사용하는 에셋 로딩 방식에 맞는 해제 API를 호출하는지

Addressables나 AssetBundle처럼 명시적 로드·해제 모델을 사용한다면, 로드한 핸들과 해제 호출을 같은 소유자가 관리해야 한다. Resources.UnloadUnusedAssets는 모든 메모리 문제를 해결하는 버튼이 아니며, 참조가 남아 있는 리소스를 강제로 정리하지 않는다. 호출 비용과 프레임 영향을 측정한 뒤 제한된 전환 구간에서만 사용한다.

## 점진적 GC와 수동 제어

Unity 6에서는 점진적 GC가 기본적으로 활성화되어 있다. 점진적 GC는 컬렉션 작업을 여러 프레임에 나누므로 긴 프레임 중단을 줄일 수 있지만, 총 작업량을 줄이지는 않는다.

| 상황 | 권장 대응 |
| --- | --- |
| GC 스파이크가 반복됨 | 먼저 할당 발생 위치를 줄이고, 점진적 GC가 활성화됐는지 확인한다 |
| 참조 변경이 매우 많음 | 점진적 작업이 효과적인지 Profiler로 검증한다 |
| 메모리 압박이 큼 | 리소스 수명과 장기 참조를 먼저 수정한다 |
| 수동 GC 제어를 고려함 | 앱 전체 정책으로만 사용하고, 명확한 중단 구간과 측정 결과를 확보한다 |

System.GC.Collect는 직접 컬렉션을 실행할 수 있으므로 일반적인 업데이트 루프나 씬 전환에 습관적으로 넣지 않는다. 점진적 GC가 지원되지 않는 대상도 있으므로, 대상 기기와 플랫폼별 프로파일링 결과를 기준으로 결정한다.

## 빌드 크기와 런타임 메모리를 혼동하지 않는다

관리되는 코드 스트리핑은 최종 빌드 크기를 줄이고 IL2CPP 변환 대상 코드량에 영향을 줄 수 있다. 하지만 이것만으로 실행 중 에셋 메모리나 관리되는 객체 누수를 해결할 수는 없다.

스트리핑 수준을 조정할 때는 리플렉션 기반 코드의 보존과 릴리스 테스트가 필요하다. 스크립팅 백엔드·스트리핑·GC의 관계는 [Unity 6 스크립팅 백엔드와 가비지 컬렉션](unity-compiler-gc.md)에서 확인한다.

## 배포 전 체크리스트

- 실제 대상 기기에서 문제 시나리오를 재현했는가
- Profiler와 두 개 이상의 Memory Profiler 스냅샷을 비교했는가
- GC.Alloc과 프레임 스파이크를 같은 구간에서 확인했는가
- 장수 객체, static 필드, 이벤트 구독이 이전 씬을 참조하지 않는가
- 에셋 로딩 방식에 맞는 해제 호출과 소유자가 명확한가
- 점진적 GC 또는 수동 GC 설정 변경 전후를 재측정했는가

## 참고 자료

- [Unity 6 Memory Profiler](https://docs.unity3d.com/cn/6000.0/Manual/com.unity.memoryprofiler.html)
- [Unity 6 MemoryProfiler API](https://docs.unity3d.com/cn/6000.0/ScriptReference/Unity.Profiling.Memory.MemoryProfiler.html)
- [Unity 6 ProfilerRecorder API](https://docs.unity3d.com/ja/6000.0/ScriptReference/Unity.Profiling.ProfilerRecorder.html)
- [Unity 6 점진적 가비지 컬렉션](https://docs.unity3d.com/kr/6000.0/Manual/performance-incremental-garbage-collection.html)
