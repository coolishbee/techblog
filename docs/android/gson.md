안드로이드에서 서버와 통신함에 있어서 Json 포맷이 필수적으로 활용되기때문에 Gson 사용법을 정리하게 되었다.
사실 서버에서 json 규격에 맞게 데이터를 정상적으로 준다면 더 아름답게 캐스팅이 가능하지 이렇게 파싱할 일이 거이 없을 것이다.
혹시라도 만약 이렇게 list(배열)을 배열로 안주고 "" string 으로 준다면 이 방법을 권합니다.

------
### json data

```json
{
   "statusCode":200,
   "list":"[
      {
         "IBNSeq":4,
         "AppId":200301,
         "StoreList":"Google/Galaxy",
         "Sort":0,
         "IsDel":false,
         "LangList":[
            {
               "Language":23,
               "Title":"afewewf",
               "Contents":"ewafaewf",
               "Images":"https://cdn.amazon.com/200301/202008/P20080703481350230.jpg",
               "Link":"awfefewewa"
            },
            {
               "Language":10,
               "Title":"ewafwef",
               "Contents":"aewfaewf",
               "Images":"https://cdn.amazon.com/200301/202008/P20080703482725303.jpg",
               "Link":"wafewefa"
            }
         ],
         "StartDate":"2020-07-16T06:00:00",
         "EndDate":"2020-11-05T06:55:00",
         "RegDate":"2020-10-13T16:44:43.463"
      },
      {
         "IBNSeq":3,
         "AppId":200301,
         "StoreList":"Google/IOS",
         "Sort":1,
         "IsDel":false,
         "LangList":[
            {
               "Language":23,
               "Title":"test",
               "Contents":"tesestagaew",
               "Images":"https://cdn.amazon.com/200301/202008/P20080703484426532.jpg",
               "Link":"urlurlurl"
            },
            {
               "Language":10,
               "Title":"fewa",
               "Contents":"eafw",
               "Images":"https://cdn.amazon.com/200301/202008/P20080703484933609.jpg",
               "Link":"urlurlurlurl"
            }
         ],
         "StartDate":"2020-07-23T06:00:00",
         "EndDate":"2020-11-29T06:15:00",
         "RegDate":"2020-10-13T16:45:00.463"
      }
   ]"
}
```

------

### 클래스 구조

```java
public class RespAppImageBanner {

    @SerializedName("IBNSeq")
    @Expose
    public int IBNSeq;

    @SerializedName("AppId")
    @Expose
    public int AppId;

    @SerializedName("StoreList")
    @Expose
    public String StoreList;

    @SerializedName("Sort")
    @Expose
    public int Sort;

    @SerializedName("IsDel")
    @Expose
    public boolean IsDel;

    @SerializedName("StartDate")
    @Expose
    public String StartDate;

    @SerializedName("EndDate")
    @Expose
    public String EndDate;

    @SerializedName("RegDate")
    @Expose
    public String RegDate;

    @SerializedName("LangList")
    @Expose
    public LangList list = new LangList();

    public class LangList {
        @SerializedName("Language")
        @Expose
        public String Language;

        @SerializedName("Title")
        @Expose
        public String Title;

        @SerializedName("Contents")
        @Expose
        public String Contents;

        @SerializedName("Images")
        @Expose
        public String Images;

        @SerializedName("Link")
        @Expose
        public String Link;
    }
}
```

### 파싱하기

```java
// response.body().toString() 이 jsonData
Gson gson = new Gson();
JsonObject info = gson.fromJson(response.body().toString(), JsonObject.class);
JsonArray listArray = gson.fromJson(info.get("list").getAsString(), JsonArray.class);

ArrayList<String> arrayList = new ArrayList<String>();
for(int i=0; i<listArray.size(); i++)
{
    JsonObject jsonObject = gson.fromJson(listArray.get(i).toString(), JsonObject.class);
    JsonArray langArray = gson.fromJson(
            jsonObject.get("LangList").getAsJsonArray(),
            JsonArray.class);

    Type listType = new TypeToken<List<RespAppImageBanner.LangList>>(){}.getType();
    List<RespAppImageBanner.LangList> langList = gson.fromJson(langArray.toString(), listType);
    for (int j=0; j<langList.size(); j++)
    {
        Log.d("http", langList.get(i).Contents);
        Log.d("http", langList.get(i).Images);
        Log.d("http", langList.get(i).Link);
        Log.d("http", langList.get(i).Title);
        Log.d("http", langList.get(i).Language);
    }
}
```