> 기본적인 정보들은 구글링했을때 워낙 정보가 많기때문에 생략하겠습니다.
>
> 레트로핏의 기본적인 예제를 해봤다는 가정하에 만든 문서입니다.



### Get API 정의하기

```java
public class RespServerTime {

    public final String utc;
    public final long unix;

    public RespServerTime(String utc, long unix) {
        this.utc = utc;
        this.unix = unix;
    }
}
```

```
@GET("/api/serverTime")
Call<RespServerTime> getServerTime();
```

```
https://baseURL.co.kr/api/serverTime
```

기본적인 get 방식이며 interface 를 위와 같이 구성했을시 호출되는 URL 입니다.

리턴값에 해당클래스로 정의하면 성공 리턴시 별도 파싱없이 캐스팅해서 사용할 수 있습니다.

------

### BaseURL 없이 FULL URL 로 Get API 호출하기

```java
public class RespAppGeo {

    public final String status;
    public final String country;
    public final String countryCode;
    public final String region;
    public final String regionName;
    public final String city;
    public final String timezone;
    public final String query;

    public RespAppGeo(String status, String country, String countryCode, String region,
                      String regionName, String city, String timezone, String query) {

        this.status = status;
        this.country = country;
        this.countryCode = countryCode;
        this.region = region;
        this.regionName = regionName;
        this.city = city;
        this.timezone = timezone;
        this.query = query;
    }

    public String toString() {
        return "RespAppGeo{" +
                "status='" + status + '\'' +
                ", country='" + country + '\'' +
                ", countryCode='" + countryCode + '\'' +
                ", region='" + region + '\'' +
                ", regionName='" + regionName +
                ", city='" + city + '\'' +
                ", timezone='" + timezone + '\'' +
                ", query='" + query + '\'' +
                '}';
    }
}
```

```
@GET
Call<RespAppGeo> getGeoAPI(@Url String url);
```

```
http://ip-api.com/json
```

API 호출시 위와 같이 Full URL 을 넣어주면 됩니다. 따로 BaseURL 영향을 받지 않고 호출하고 싶을때 사용하면 좋습니다.

------

### POST API 정의하기

```java
public class ReqAppInfo {

    @SerializedName("table")
    @Expose
    public String table;

    @SerializedName("data")
    @Expose
    public AppData data = new AppData();

    public class AppData {

        @SerializedName("projectId")
        @Expose
        public int projectId;

        @SerializedName("appPackage")
        @Expose
        public String appPackage;
    }
}
```

```java
public class RespAppInfo {    

    @SerializedName("AppLanguage")
    @Expose
    public String AppLanguage;    

    @SerializedName("AppAuthKey_Google")
    @Expose
    public String AppAuthKey_Google;

    @SerializedName("AppAuthKey_Facebook")
    @Expose
    public String AppAuthKey_Facebook;

    @SerializedName("AppAuthKey_Apple")
    @Expose
    public String AppAuthKey_Apple;

}
```

```
@POST("/api/Info")
Call<RespAppInfo> postAppInfo(@Query("t") long time, @Query("sign") String sign,
                             @Body ReqAppInfo data);
```

```
https://baseURL.co.kr/api/Info?t={0}&sign={1}
```

기본적인 post 방식이며 interface 를 위와 같이 구성했을시 호출되는 URL 입니다.

마찬가지로 리턴값에 Resp클래스나 JsonObject로 정의하면 성공 리턴시 파싱없이 캐스팅해서 사용할 수 도 있고

JsonObject 같은 경우에는 Gson 으로 파싱할 수 있습니다.

------

### RetrofitClient 구현부

```java
public class RetrofitClient {

    public RetrofitApiService apiService;
    private static Retrofit retrofit;


    private static class SingletonHolder {
        private static RetrofitClient INSTANCE = new RetrofitClient(RetrofitUtils.API_SERVER_Base_URL);
    }

    public static RetrofitClient get() {
        return SingletonHolder.INSTANCE;
    }

    public RetrofitClient(@NonNull final String apiBaseUrl) {
        retrofit = new Retrofit.Builder()
                .baseUrl(apiBaseUrl)
                .addConverterFactory(GsonConverterFactory.create())
                .client(RetrofitUtils.getUnsafeOkHttpClient().build())
                .build();

        apiService = create(RetrofitApiService.class);
    }

    /**
     * create you ApiService
     * Create an implementation of the API endpoints defined by the {@code service} interface.
     */
    private  <T> T create(final Class<T> service) {
        if(service == null) {
            throw new RuntimeException("Api service is null");
        }
        return retrofit.create(service);
    }    
}
```

### RetrofitClient 사용하기

```java
Call<RespServerTime> getServerTime = RetrofitClient.get().apiService.getServerTime();
getServerTime.enqueue(new Callback<RespServerTime>() {
    @Override
    public void onResponse(@NotNull Call<RespServerTime> call,
                            @NotNull Response<RespServerTime> response) {
        RespServerTime respData = response.body();        
    }

    @Override
    public void onFailure(@NotNull Call<RespServerTime> call, @NotNull Throwable t) {
        
    }
});
```

> [retrofitExample 보러가기](https://github.com/james-chun-dev/retrofitExample)