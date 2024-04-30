---
comments: true
---

# GooglePlay 결제 시스템

## Google Play Billing Library Version 3

### 참고사항

* 테스트결제시 apk 빌드는 develop 인증서 사용가능. (단 프로덕션 이하로 앱게시 필요)
* 결제테스트시 스토어 관리자계정이 아닌 다른 구글계정으로 테스트 필요.
* 최신 빌링 라이브러리 버전.
* AndroidManifest.xml에 BILLING 권한 필요없음.

### 예제

```groovy
dependencies {
    implementation 'com.android.billingclient:billing:3.0.3'
}
```

!!! example

    === "JAVA"
        ```java
        public class GoogleBillingImpl implements PurchasesUpdatedListener {
            private static final String TAG = "GoogleBillingImpl";

            private final BillingClient mBillingClient;
            private List<SkuDetails> skuDetailsList = new ArrayList<>();        

            public GoogleBillingImpl(@NonNull final Context applicationContext)
            {        
                mBillingClient = BillingClient.newBuilder(applicationContext)
                        .enablePendingPurchases()
                        .setListener(this)
                        .build();
            }
            
            public void init()
            {
                mBillingClient.startConnection(new BillingClientStateListener() {
                    @Override
                    public void onBillingSetupFinished(@NonNull BillingResult billingResult)
                    {
                        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK)
                        {
                            // The billing client is ready. You can query purchases here.
                            List<String> strList = new ArrayList<>();
                            strList.add("test_1000");
                            strList.add("test_2000");

                            SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
                            params.setSkusList(strList).setType(BillingClient.SkuType.INAPP);

                            mBillingClient.querySkuDetailsAsync(params.build(), new SkuDetailsResponseListener() {
                                @Override
                                public void onSkuDetailsResponse(@NonNull BillingResult billingResult, 
                                                                @Nullable List<SkuDetails> list)
                                {
                                    // Process the result.
                                    if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK
                                            && list != null)
                                    {
                                        if(list.isEmpty())
                                        {                                    
                                            Log.d(TAG, "list is zero");
                                        }else{
                                            skuDetailsList = list;
                                        }
                                    }
                                }
                            });
                        }else{
                            Log.d(TAG, "google purchase error");                    
                        }
                    }

                    @Override
                    public void onBillingServiceDisconnected() {
                        // Try to restart the connection on the next request to
                        // Google Play by calling the startConnection() method.
                        Log.d(TAG, "onBillingServiceDisconnected");
                    }
                });
            }

            public void purchase(Activity activity, String productId) 
            {
                BillingFlowParams flowParams = null;
                BillingResult billingResult;

                SkuDetails sku = getSkuDetail(productId);
                if( sku != null )
                {
                    flowParams = BillingFlowParams.newBuilder()
                            .setSkuDetails(sku)
                            .build();
                    billingResult = mBillingClient.launchBillingFlow(activity, flowParams);
                }
                else
                {
                    Log.d(TAG, "sku is null");
                }
            }

            @Override
            public void onPurchasesUpdated(BillingResult billingResult, @Nullable List<Purchase> list) {
                if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK
                        && list != null) {
                    for (Purchase purchase : list) {
                        handlePurchase(purchase);
                    }
                } else if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.USER_CANCELED) {
                    // Handle an error caused by a user cancelling the purchase flow.
                    Log.d(TAG, "user purchase cancel");
                } else {
                    // Handle any other error codes.
                    Log.d(TAG, "google purchase error");
                }
            }

            private void handlePurchase(Purchase purchase) {
                String purchaseToken, payLoad;
                purchaseToken = purchase.getPurchaseToken();
                payLoad = purchase.getDeveloperPayload();

                if(purchase.getPurchaseState() == Purchase.PurchaseState.PURCHASED)
                {
                    ConsumeParams consumeParams =
                        ConsumeParams.newBuilder()
                                .setPurchaseToken(purchaseToken)                        
                                .build();

                    mBillingClient.consumeAsync(consumeParams, new ConsumeResponseListener() {
                        @Override
                        public void onConsumeResponse(@NonNull BillingResult billingResult, @NonNull String s)
                        {
                            if(billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) 
                            {                                                
                                Log.d(TAG, "google purchase success");                    
                            }else{
                                Log.d(TAG, "google purchase consume error");
                            }
                        }
                    });
                }        
            }

            private SkuDetails getSkuDetail(String productId) {
                for(SkuDetails item : skuDetailsList) {
                    if(item.getSku().equals(productId)) {
                        return item;
                    }
                }
                return null;
            }    
        }
        ```

    === "Kotlin"

        ``` kotlin
        class BillingGoogleImpl(
            activity: Activity
        ) : PurchasesUpdatedListener {

            private val skuDetailsList: MutableList<SkuDetails> = arrayListOf()

            private var billingClient = BillingClient.newBuilder(activity)
                .setListener(this)
                .enablePendingPurchases()
                .build()

            fun init(productList: List<String>
            ) {
                billingClient.startConnection(object : BillingClientStateListener {
                    override fun onBillingSetupFinished(billingResult: BillingResult) {
                        if(billingResult.responseCode == BillingClient.BillingResponseCode.OK)
                        {
                            queryProductDetailsAsync(productList)
                        }else{
                            Log.d(TAG, billingResult.toString())
                        }
                    }

                    override fun onBillingServiceDisconnected() {
                        Log.d(TAG, "onBillingServiceDisconnected")
                    }

                })
            }
            
            fun queryProductDetailsAsync(productList: List<String>
            ) {
                val params = SkuDetailsParams.newBuilder()
                params.setSkusList(productList).setType(BillingClient.SkuType.INAPP)

                billingClient.querySkuDetailsAsync(params.build()) { billingResult, skuDetailsResult ->
                    if(billingResult.responseCode == BillingClient.BillingResponseCode.OK
                        && skuDetailsResult != null) {
                        for (skuDetail in skuDetailsResult)
                        {
                            Log.d(TAG, skuDetail.toString())
                            skuDetailsList.add(skuDetail)                    
                        }
                    }else{
                        Log.d(TAG, billingResult.toString())
                    }
                }
            }

            override fun onPurchasesUpdated(billingResult: BillingResult,
                                            list: List<Purchase>?)
            {
                if(billingResult.responseCode == BillingClient.BillingResponseCode.OK
                    && list != null) {
                    for (purchase in list){
                        consumeAsync(purchase)
                    }
                }else{
                    Log.d(TAG, billingResult.toString())
                }
            }

            fun purchase(activity: Activity,
                                productId: String                          
            ) {
                val sku = getSkuDetail(productId)
                if(sku != null){
                    val flowParams = BillingFlowParams.newBuilder()
                        .setSkuDetails(sku)
                        .build()
                    val responseCode = billingClient.launchBillingFlow(activity, flowParams).responseCode
                }else{
                    // sku null
                }

            }

            private fun consumeAsync(purchase: Purchase) {
                val consumeParams =
                    ConsumeParams.newBuilder()
                        .setPurchaseToken(purchase.purchaseToken)
                        .build()

                billingClient.consumeAsync(consumeParams) { billingResult, purchaseToken ->
                    if(billingResult.responseCode == BillingClient.BillingResponseCode.OK){

                        Log.d(TAG, purchase.originalJson)
                        Log.d(TAG, purchaseToken)                
                    }else{
                        Log.d(TAG, billingResult.toString())
                    }
                }
            }    

            fun getPrice(productId: String) : Float {
                for (item in skuDetailsList) {
                    if (item.sku == productId) {
                        val price = item.priceAmountMicros / 1000000.0
                        return (round(price*100)/100).toFloat()
                    }
                }
                return 0f
            }

            fun getCurrencyCode(productId: String): String {
                for (item in skuDetailsList) {
                    if (item.sku == productId) {
                        return item.priceCurrencyCode
                    }
                }
                return ""
            }

            fun getSkuDetail(pid: String) : SkuDetails? {
                for (item in skuDetailsList) {
                    if (item.sku == pid) {
                        return item
                    }
                }
                return null
            }

            companion object {
                private const val TAG = "BillingGoogleImpl"
            }
        }
        ```