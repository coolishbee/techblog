
## string to enum 문자열을 열거형으로 변환

https://magicmon.tistory.com/102

https://www.delftstack.com/ko/howto/csharp/how-to-convert-string-to-enum-in-csharp/


## extension method

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


