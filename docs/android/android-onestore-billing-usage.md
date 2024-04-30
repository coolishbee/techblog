---
comments: true
---

# OneStore 결제 시스템

## SDK v19 API v6

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

    === "Kotlin"

        ```kotlin
        class BillingOneStoreImpl(
            private val activity: Activity
        ): PurchasesUpdatedListener {    

            private var skuDetailsList: listOf<ProductDetail>()
            private val billingClient = PurchaseClient.newBuilder(activity)
                .setListener(this)
                .build()

            fun init(productList: List<String>) {

                val signInClient = GaaSignInClient.getClient(activity)
                signInClient.silentSignIn { backgroundResult ->
                    // Onestore login failed
                    if (!backgroundResult.isSuccessful) {
                        // error log
                        signInClient.launchSignInFlow(activity) { foregroundResult ->
                            if (!foregroundResult.isSuccessful) {
                                // error log
                            }
                        }else {
                            queryProductDetailsAsync(productList)
                        }
                    }
                    // Onestore login succeeded
                    else {
                        billingClient.startConnection(object : PurchaseClientStateListener {
                            override fun onSetupFinished(iapResult: IapResult) {
                                if (iapResult.isSuccess) {
                                    queryProductDetailsAsync(productList)
                                }                        
                            }
                            override fun onServiceDisconnected() {                        
                            }
                        })
                    }
                }
            }

            fun queryProductDetailsAsync(productList: List<String>) {        

                val params = ProductDetailsParams.newBuilder()
                    .setProductIdList(productList)
                    .setProductType(ProductType.INAPP)
                    .build()

                billingClient.queryProductDetailsAsync(params) { iapResult, productDetails ->
                    if (iapResult.isSuccess) {
                        skuDetailsList = productDetails                
                    }            
                }
            }

            fun purchase(activity: Activity,
                        productId: String                          
            ) {
                val product = getProductDetail(productId)
                if (product != null){
                    val flowParams = PurchaseFlowParams.newBuilder()
                        .setProductId(productId)
                        .setProductType(ProductType.INAPP)
                        .build()

                    billingClient.launchPurchaseFlow(activity, flowParams)
                }        
            }

            override fun onPurchasesUpdated(iapResult: IapResult,
                                            list: MutableList<PurchaseData>?)
            {
                if (iapResult.isSuccess && list != null){
                    for (purchase in list){
                        consumeAsync(purchase)
                    }
                }       
            }

            private fun consumeAsync(purchase: PurchaseData)
            {
                val consumeParams = ConsumeParams.newBuilder()
                    .setPurchaseData(purchase)
                    .build()

                billingClient.consumeAsync(consumeParams) { billingResult, purchaseData ->
                    if (billingResult.isSuccess) {

                        Log.d(TAG, purchaseData.originalJson)
                        Log.d(TAG, purchaseData.purchaseToken)                
                    }            
                }
            }    

            private fun getProductDetail(marketProductId: String) : ProductDetail? {        
                for (item in skuDetailsList){
                    if (item.productId == marketProductId){
                        return item
                    }
                }
                return null
            }

            companion object {
                    private const val TAG = "BillingOneStoreImpl"
            }
        }
        ```