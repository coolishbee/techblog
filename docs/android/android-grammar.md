---
comments: true
---

## aes256 암호화 복호화

!!! cbc사용
    === "PKCS5Padding"

        ```kotlin
        fun encryptCBC(strToEncrypt: String, key: String, iv: String): String {
            try {

                val ivParameterSpec = IvParameterSpec(iv.toByteArray())
                val secretKey = SecretKeySpec(key.toByteArray(), "AES")

                val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
                cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec)

                val crypted = cipher.doFinal(strToEncrypt.toByteArray())
                val encodedByte = Base64.encode(crypted, Base64.DEFAULT)

                return String(encodedByte)

            } catch (e: Exception) {
                Log.e("Exception", "Exception : ${e.message}")
            }
            return ""
        }

        fun decryptCBC(strToDecrypt: String, key: String, iv: String): String {

            val ivParameterSpec = IvParameterSpec(iv.toByteArray())
            val secretKey = SecretKeySpec(key.toByteArray(), "AES")

            val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
            cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec)

            val decodedByte = Base64.decode(strToDecrypt, Base64.DEFAULT)
            val byteResult = cipher.doFinal(decodedByte)

            return String(byteResult)
        }
        ```

    === "PKCS7Padding"

        ```kotlin
        object AESEncyption {

            const val secretKey = "tK5UTui+DPh8lIlBxya5XVsmeDCoUl6vHhdIESMB6sQ="
            const val salt = "QWlGNHNhMTJTQWZ2bGhpV3U=" // base64 decode => AiF4sa12SAfvlhiWu
            const val iv = "bVQzNFNhRkQ1Njc4UUFaWA==" // base64 decode => mT34SaFD5678QAZX

            fun encrypt(strToEncrypt: String) : String?
            {
                try
                {
                    val ivParameterSpec = IvParameterSpec(Base64.decode(iv, Base64.DEFAULT))

                    val factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1")
                    val spec =  PBEKeySpec(secretKey.toCharArray(), Base64.decode(salt, Base64.DEFAULT), 10000, 256)
                    val tmp = factory.generateSecret(spec)
                    val secretKey =  SecretKeySpec(tmp.encoded, "AES")

                    val cipher = Cipher.getInstance("AES/CBC/PKCS7Padding")
                    cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec)
                    return Base64.encodeToString(cipher.doFinal(strToEncrypt.toByteArray(Charsets.UTF_8)), Base64.DEFAULT)
                }
                catch (e: Exception)
                {
                    println("Error while encrypting: $e")
                }
                return null
            }

            fun decrypt(strToDecrypt : String) : String? {
                try
                {
                    val ivParameterSpec =  IvParameterSpec(Base64.decode(iv, Base64.DEFAULT))

                    val factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1")
                    val spec =  PBEKeySpec(secretKey.toCharArray(), Base64.decode(salt, Base64.DEFAULT), 10000, 256)
                    val tmp = factory.generateSecret(spec);
                    val secretKey =  SecretKeySpec(tmp.encoded, "AES")

                    val cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");
                    cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
                    return  String(cipher.doFinal(Base64.decode(strToDecrypt, Base64.DEFAULT)))
                }
                catch (e : Exception) {
                    println("Error while decrypting: $e");
                }
                return null
            }
        }
        ```

!!! gcm사용
    === "encrypt"

        ```kotlin
        fun encrypt(password: String): String {
            val secretKeySpec = SecretKeySpec(password.toByteArray(), "AES")
            val iv = ByteArray(16)
            val charArray = password.toCharArray()
            for (i in 0 until charArray.size){
                iv[i] = charArray[i].toByte()
            }
            val ivParameterSpec = IvParameterSpec(iv)

            val cipher = Cipher.getInstance("AES/GCM/NoPadding")
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, ivParameterSpec)

            val encryptedValue = cipher.doFinal(this.toByteArray())
            return Base64.encodeToString(encryptedValue, Base64.DEFAULT)
        }
        ```

    === "decrypt"

        ```kotlin
        fun decrypt(password: String): String {
            val secretKeySpec = SecretKeySpec(password.toByteArray(), "AES")
            val iv = ByteArray(16)
            val charArray = password.toCharArray()
            for (i in 0 until charArray.size){
                iv[i] = charArray[i].toByte()
            }
            val ivParameterSpec = IvParameterSpec(iv)

            val cipher = Cipher.getInstance("AES/GCM/NoPadding")
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivParameterSpec)

            val decryptedByteValue = cipher.doFinal(Base64.decode(this, Base64.DEFAULT))
            return String(decryptedByteValue)
        }
        ```

## 구글 인앱결제시 priceAmountMicros 에서 Float 으로 변환

SkuDetails.priceAmountMicros 의 대한 설명:
![priceAmountMicros](https://velog.velcdn.com/images/james-chun-dev/post/6545b809-bf0f-4583-89e7-d99e6aaf237a/image.png)

priceAmountMicros to float 으로 변환후 소수점 3번째짜리에서 반올림

=== "JAVA"

    ``` java
    public float getPrice(SkuDetails item) {
        double price = item.getPriceAmountMicros() / 1000000.0;
        return (float) (Math.round(price*1e2)/1e2);
    }
    ```

=== "Kotlin"

    ``` kotlin
    fun getPrice(item: SkuDetails) : Float {
        val price = item.priceAmountMicros / 1000000.0
        return (round(price*100)/100).toFloat()
    }    
    ```

## context extension

!!! 유틸함수
    === "앱이 설치되었는지 판단하는 함수"

        ```kotlin
        fun Context.isInstalledApp(packageName: String): Boolean {
            val intent = packageManager.getLaunchIntentForPackage(packageName)
            return intent != null
        }
        ```

    === "특정 앱을 실행하는 함수"

        ```kotlin
        fun Context.openApp(packageName: String) {
            val intent = packageManager.getLaunchIntentForPackage(packageName)
            startActivity(intent)
        }
        ```

    === "마켓으로 이동하는 함수"

        ```kotlin
        fun Context.market(packageName: String): Boolean {
            return try {
                val intent = Intent(Intent.ACTION_VIEW)
                intent.data = Uri.parse("market://details?id=$packageName")
                startActivity(intent)
                true
            } catch (e: ActivityNotFoundException) {
                e.printStackTrace()
                false
            }
        }
        ```

## 문자열내에 $ 출력하기

```kotlin
const val HashKey = "\$"
```

## Android Language Code

- zh_cn (간체)
- zh_tw (번체)

!!! Reference
        [https://developers.google.com/interactive-media-ads/docs/sdks/android/client-side/localization](https://developers.google.com/interactive-media-ads/docs/sdks/android/client-side/localization)

## enum 재정의

```java
public enum ApiResponseCode {

    /**
     * The request was successful.
     */
    SUCCESS(1000),
    /**
     * The request was canceled.
     */
    CANCEL(1001),
    /**
     * A network error occurred.
     */
    NETWORK_ERROR(1002);

    int value;

    ApiResponseCode(int value) {
        this.value = value;
    }
    public int getValue() {
        return value;
    }
}
```

## 문자열 분리

```java
String lang = "kr/en/jp";
String[] arr = lang.split("/");
```

## md5, sha256 로 암호화

!!! md5 로 암호화
    === "JAVA"

        ``` java
        public static String md5(String str){
            String MD5 = "";
            try{
                MessageDigest md = MessageDigest.getInstance("MD5");
                md.update(str.getBytes("UTF-8"));
                byte byteData[] = md.digest();
                StringBuffer sb = new StringBuffer();
                for(int i = 0 ; i < byteData.length ; i++)
                    sb.append(Integer.toString((byteData[i]&0xff) + 0x100, 16).substring(1));
                MD5 = sb.toString();
            }
            catch(NoSuchAlgorithmException e) { e.printStackTrace(); MD5 = null; }
            catch (UnsupportedEncodingException e) { e.printStackTrace(); MD5 = null; }
            return MD5;
        }
        ```

    === "Kotlin"

        ``` kotlin
        private fun md5(input:String): String {
            val md = MessageDigest.getInstance("MD5")
            return BigInteger(1, md.digest(input.toByteArray())).toString(16).padStart(32, '0')
        }
        ```

!!! sha256 로 암호화
    === "JAVA"

        ```java
        public static String sha256(String str) {
            String SHA = "";
            try{
                MessageDigest sh = MessageDigest.getInstance("SHA-256");
                sh.update(str.getBytes());
                byte byteData[] = sh.digest();
                StringBuffer sb = new StringBuffer();
                for(int i = 0 ; i < byteData.length ; i++)
                    sb.append(Integer.toString((byteData[i]&0xff) + 0x100, 16).substring(1));
                SHA = sb.toString();
            }catch(NoSuchAlgorithmException e) { e.printStackTrace(); SHA = null; }
            return SHA;
        }
        ```

    === "Kotlin"

        ```kotlin
        fun sha256(s: String): String {
            val md = MessageDigest.getInstance("SHA-256")
            val digest = md.digest(s.toByteArray())
            val hash = StringBuilder()
            for (c in digest) {
                hash.append(String.format("%02x", c))
            }
            return hash.toString()
        }
        ```

!!! Reference
        [https://stackoverflow.com/questions/64171624/how-to-generate-an-md5-hash-in-kotlin](https://stackoverflow.com/questions/64171624/how-to-generate-an-md5-hash-in-kotlin)

## UTC Time 가져오기

```java
private static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss.SSSSSSS";

public static Date getUtcDatetimeAsDate()
{
    return stringDateToDate(getUtcDatetimeAsString());
}
// UTC Now Time get
public static String getUtcDatetimeAsString()
{
    final SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
    sdf.setTimeZone(TimeZone.getTimeZone("UTC"));
    final String utcTime = sdf.format(new Date());

    return utcTime;
}
// DATE_FORMAT 형태의 string 을 Date 로 return
public static Date stringDateToDate(String strDate)
{
    Date dateToReturn = null;
    SimpleDateFormat dateFormat = new SimpleDateFormat(DATE_FORMAT);

    try{
        dateToReturn = (Date)dateFormat.parse(strDate);
    }catch (ParseException e) {
        e.printStackTrace();
    }

    return dateToReturn;
}
```

## millisecond to Date, String 로 변환

```java
private static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";

//long형 타임을 String으로 변환.
public static String longTimeToDatetimeAsString(long resultTime)
{
    SimpleDateFormat dateFormat = new SimpleDateFormat(DATE_FORMAT);
    String formatTime = dateFormat.format(resultTime);
    return formatTime;
}
//long형 타임을 Date 로 변환.
public static Date setTimeConvertDate(long time)
{
    Date unixDate = null;
    SimpleDateFormat dateFormat = new SimpleDateFormat(DATE_FORMAT);
    String formatTime = dateFormat.format(time);
    unixDate = stringDateToDate(formatTime);

    return unixDate;
}
// 서버에서 주는 utc string을 getTime형태로 변환 후 현재시간과 차로 offset 계산.
public static long serverUTCParse(String utc)
{
    Date serverDate = null;
    SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);

    SimpleDateFormat dateFormat = new SimpleDateFormat(DATE_FORMAT);
    dateFormat.setTimeZone(TimeZone.getTimeZone("utc"));
    String formatTime = null;

    try{
        serverDate = (Date)sdf.parse(utc);
        formatTime = dateFormat.format(serverDate.getTime());

        serverDate = stringDateToDate(formatTime);
    }catch (ParseException e) {
        e.printStackTrace();;
    }

    long timeOffset = serverDate.getTime() - getUtcDatetimeAsDate().getTime(); //서버시간-현재시간

    return timeOffset;
}
```

## Timezone

``` java
public static String getTimeZone() {

        TimeZone timeZone = TimeZone.getDefault();
        Calendar calendar = GregorianCalendar.getInstance(timeZone);
        int offsetInMillis = timeZone.getOffset(calendar.getTimeInMillis());

        Log.v("gmt offset(sec)", String.valueOf(offsetInMillis));

        //초에서 분으로 환산
        String offset = String.format(Locale.getDefault(),"%02d",
                Math.abs((offsetInMillis / 60000)));
        Log.v("gmt offset(min)", offset);
        offset = (offsetInMillis >= 0 ? "+" : "-") + offset;

        String strGmt = timeZone.getDisplayName(false,TimeZone.SHORT);
        String strId = timeZone.getID();

        String strTimeZone = String.format("Local;%s;(%s) Local Time;%s;", offset, strGmt, strId);

        return strTimeZone;
}
//gmt offset(sec): 32400000
//gmt offset(min): 540
//Local;+540;(GMT+09:00) Local Time;Asia/Seoul;
```

## UUID 생성하기

``` java
public static String getUUID() {        
        return UUID.randomUUID().toString();
}
//output : 2b2204c3-762f-4aae-8d50-241ad7a2dbae
```

## DeviceName 가져오기

``` java
public static String getDeviceModel() {
        return String.format("%s %s", Build.BRAND, Build.MODEL);
}
```

## DeviceID 가져오기

유니티엔진에서 제공하는 [deviceUniqueIdentifier](https://docs.unity3d.com/ScriptReference/SystemInfo-deviceUniqueIdentifier.html) 와 동일하게 만들어줍니다. ([md5 함수](#md5-sha256)로 이동)

``` java
public static String getDeviceId(Context context) {   
   String deviceId = "";

   String android_id = Settings.Secure.getString(
            context.getContentResolver(),
            Settings.Secure.ANDROID_ID);
   deviceId = md5Encode(android_id);

   return deviceId;
}
```

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