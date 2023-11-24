
## 문자열을 열거형으로 변환

=== "c# 7.0"

    ```c#
    StatusEnum myStatus;
    Enum.TryParse("Active", out StatusEnum myStatus);
    ```
=== "c# .NET 4.0 이상"

    ```c#
    StatusEnum myStatus;
    Enum.TryParse("Active", out myStatus);
    ```

=== "c# .NET 4.0 이전"

    ```c#
    StatusEnum MyStatus = (StatusEnum)Enum.Parse(typeof(StatusEnum), "Active");
    ```

!!! Reference

    https://stackoverflow.com/questions/16100/convert-a-string-to-an-enum-in-c-sharp


## 확장 메서드

예제를 작성하기전에 간단한 설명을 덧붙히자면 다른 언어들의 익스텐션과 동일한 방식이다. Extension 뜻이 의미하는 것처럼 기존 클래스의 멤버함수를 직접 커스텀 확장하는 개념이다.

!!! example

    === "definition"

        ```c#
        using UnityEngine;

        namespace TestSDK
        {
            public static class ExtensionMethods
            {
                public static void Log(this object value)
                {
                    if (TestSDKSettings.DevBuild)
                        Debug.Log(value.ToString());
                }        
            }
        }
        ```

    === "usage"

        ```c#
        public void TestCall(string result)
        {
            result.Log();
        }
        ```

이렇게 하면 모든 프로젝트 영역에 Debug.Log 조건문을 걸지 않아도 된다.


## c++ 과 c# 의 차이

컴파일시 c++은 기계어로 변환되고 c#은 [중간언어](https://learn.microsoft.com/ko-kr/dotnet/standard/managed-code#intermediate-language--execution)로 변환되며<br> 실행시에 실시간으로 기계어로 번역되어 실행됩니다.

!!! tip

    c#은 코드가 실행되는 시점이고 c++은 컴파일할때 변환이 끝납니다.<br>
    즉, 컴파일 단계 중 하나인 코드생성 타이밍이 다릅니다.

- c/c++에서 사용되는 포인터는 힙 영역에 메모리를 내가 원하는 만큼 할당 할 수 있습니다. 반드시 할당했으면 해제를 해줘야 합니다.
- 포인터는 물리적 주소 번지를 이용하여 내가 원하는 물리적 주소에 접근하여 메모리가 가리키는 값을 읽을 수 있습니다.

그리고 c++에서 사용되는 레퍼런스는 포인터와 약간 다릅니다. 레퍼런스는 별칭자의 개념으로 그 메모리에 접근 할 수 있는 이름을 하나 더 생성합니다.
포인터와 같이 특정 물리적 주소를 가리키고 있어 값에 의한 접근이 아닌 참조에 의한 접근으로 메모리에 접근합니다. 
주로 클래스의 별칭으로 사용됩니다.

그러나 c#에서의 참조 타입은 클래스, 배열, 스트링, 인터페이스 등이 참조 타입입니다. 
참조 타입은 힙에 [할당](https://learn.microsoft.com/ko-kr/dotnet/standard/automatic-memory-management#allocating-memory)되며, 해당 힙 영역을 참조하지 않을 경우 3개의 세대를 거쳐 가비지컬렉터에 의해 자동으로 [수거](https://learn.microsoft.com/ko-kr/dotnet/standard/automatic-memory-management#releasing-memory)됩니다.

- c#의 참조 타입은 포인터나 레퍼런스와 같이 물리적 주소를 가리키며, 값에 의한 복사를 하지 않습니다.
- 포인터와 같이 다른 참조 객체를 가리킬 수 있지만, 번지 주소를 이용한 특정 물리적 주소의 접근은 할 수 없습니다.
- 레퍼런스와 같이 .연산자를 이용하여 메모리에 접근하지만, c#의 참조 타입은 상수 포인터가 아니므로 다른 참조 객체를 가리 킬 수 있습니다.


