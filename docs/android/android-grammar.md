
For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## string to enum 문자열을 열거형으로 변환

!!! example
    === "JAVA"

        ``` java
        public enum MyEnum {
            EnumValue1,
            EnumValue2;

            public static MyEnum fromInteger(int x) {
                switch(x) {
                case 0:
                    return EnumValue1;
                case 1:
                    return EnumValue2;
                }
                return null;
            }
        }
        ```

    === "Kotlin"

        ``` kotlin
        enum class Types(val value: Int) {
            FOO(1),
            BAR(2),
            FOO_BAR(3);

            companion object {
                fun fromInt(value: Int) = Types.values().first { it.value == value }
            }
        }
        ```

    !!! Reference
        [https://stackoverflow.com/questions/5878952/cast-int-to-enum-in-java](https://stackoverflow.com/questions/5878952/cast-int-to-enum-in-java)
        
        [https://stackoverflow.com/questions/53523948/how-do-i-create-an-enum-from-an-int-in-kotlin](https://stackoverflow.com/questions/53523948/how-do-i-create-an-enum-from-an-int-in-kotlin)


## 절대값 구하기

=== "JAVA"

    ``` java
    Math.abs(int a);
    ```

=== "Kotlin"

    ``` kotlin
    Math.abs(int a);
    ```

## 국가코드 및 국가명 가져오기

=== "JAVA"

    ``` java
    public static String getCountry() {
        return Locale.getDefault().getCountry();
    }
    public static String getDisplayCountry() {
            return Locale.getDefault().getDisplayCountry(Locale.US);
    }
    ```

=== "Kotlin"

    ``` kotlin
    fun getCountry(): String {    
        return Locale.getDefault().country
    }
    fun getDisplayCountry(): String {    
        return Locale.getDefault().displayLanguage
    }
    ```

## 언어코드 가져오기

=== "JAVA"

    ``` java
    public static String getLanguage() {
        return Locale.getDefault().getLanguage();
    }
    ```

=== "Kotlin"

    ``` kotlin
    fun getLanguage(): String {    
        return Locale.getDefault().language
    }    
    ```