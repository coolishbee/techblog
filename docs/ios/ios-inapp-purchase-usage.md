apple Developer console 에 앱 등록이 되어 있다는 가정하에 설명한다.

우선 인앱상품등록 부터 준비한다. (앱 업로드는 하지 않아도 샌드박스 결제테스트가 가능하다.)



## 인앱 상품 등록

* 소스코드단에서 제품 ID 를 사용하기 때문에 잘 정해주면 된다.
* 나머지는 수정이 가능하나 제품 ID 는 변경이 안된다.
* 등록된 상품은 수정 및 삭제가 가능하다.
* 상품이 등록되었다면 앱 업로드 또는 출시를 하지 않아도 결제테스트 준비가 끝난 것이다.



![inapp register](https://user-images.githubusercontent.com/20632507/108959794-41818780-76b8-11eb-9b00-ec904ad07bba.png)

![inapp regist](https://user-images.githubusercontent.com/20632507/108959807-48a89580-76b8-11eb-8335-a23b43f91600.png)



## 샌드박스 테스터 추가

* 샌드박스 결제테스트를 위해 테스터 계정을 추가한다.
* App Store Connect > 사용자 및 액서스 > Sandbox > 테스터 > 신규 사용자 등록
* 이메일은 애플계정에 등록되지 않은 이메일주소여야 한다.



## 인앱 결제 기능 활성화

* 프로젝트 Targets > Signing & Capabilities 탭에 가면 + Capability 버튼 클릭
* In-App Purchase 활성화를 시켜주면 자동으로 Storekit 라이브러리도 포함된다.



여기까지 인앱결제 개발을 위한 준비가 끝났다.



## 코드 구현

* setupBilling 에 NSArray (상품ID 목록) 을 넣어주면 인앱결제 모듈이 초기화된다.
* 결제가 완료된 후에는 따로 델레게이트나 block 으로 콜백을 만들어서 사용하면 된다.



BillingManager.h

```objective-c
#import <Foundation/Foundation.h>
#import <StoreKit/StoreKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface BillingManager : NSObject<SKProductsRequestDelegate,SKPaymentTransactionObserver>
{
    NSString *productID;
}

+ (instancetype)getInstance;

@property (nonatomic) NSArray *productList;

- (void)setupBilling:(NSArray*)arr;
- (void)purchase:(NSString*)pid;
- (BOOL)canMakePurchases;
- (float)getPrice:(NSString*)pid;
- (NSString*)getPriceCurrencyCode:(NSString*)pid;
- (SKProduct*)getSKProduct:(NSString*)pid;

@end

NS_ASSUME_NONNULL_END
```

BillingManager.m

```objective-c
#import "BillingManager.h"

@implementation BillingManager

+ (instancetype)getInstance {
    static BillingManager *shared = nil;
        
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        shared = [[BillingManager alloc] init];
    });
    return shared;
}

- (void)setupBilling:(NSArray *)arr
{
    _productList = arr;
    
    NSMutableArray* tempArr = [NSMutableArray array];
    for(RespAppInApp *item in _productList)
    {
        [tempArr addObject:item.ProductID];
    }
    
    NSSet *productIdentifiers = [NSSet setWithArray:tempArr];
    SKProductsRequest *productsRequest = [[SKProductsRequest alloc] initWithProductIdentifiers:productIdentifiers];
    productsRequest.delegate = self;
    [productsRequest start];
}

- (float)getPrice:(NSString *)pid
{
    for(SKProduct *skProduct in _productList)
    {
        if ([skProduct.productIdentifier isEqualToString:pid])
        {
            return [skProduct.price floatValue];
        }
    }
    return 0;
}

- (NSString*)getPriceCurrencyCode:(NSString *)pid
{
    for(SKProduct *skProduct in _productList)
    {
        if ([skProduct.productIdentifier isEqualToString:pid])
        {
            return skProduct.priceLocale.currencyCode;
        }
    }
    return @"";
}

- (SKProduct*)getSKProduct:(NSString *)pid
{
    for(SKProduct *item in _productList)
    {
        if([item.productIdentifier isEqualToString:pid] == YES)
            return item;
    }
    return nil;
}

- (BOOL)canMakePurchases {
    return [SKPaymentQueue canMakePayments];
}

- (void)purchase:(NSString *)pid
{
    productID = pid;
    if (_productList != nil)
    {
        if ([self canMakePurchases])
        {
            SKProduct *skProduct = [self getSKProduct:pid];
            if(skProduct != nil)
            {
                SKPayment *skPayment = [SKPayment paymentWithProduct:skProduct];
                [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
                [[SKPaymentQueue defaultQueue] addPayment:skPayment];
            }else{
                NSLog(@"Product is null");
            }
        } else {
            NSLog(@"No Purchases");
        }
    }
}

- (NSString *)getReceiptCode
{
    NSURL *receiptURL = [[NSBundle mainBundle] appStoreReceiptURL];
    NSData *receipt = [NSData dataWithContentsOfURL:receiptURL];

    if (!receipt) {
        NSLog(@"no receipt");
        return nil;
    } else {
        // Get the receipt in encoded format
        NSString *encodedReceipt = [receipt base64EncodedStringWithOptions:0];
        
        return encodedReceipt;
    }
}

#pragma mark - StoreKit Delegate
- (void)paymentQueue:(nonnull SKPaymentQueue *)queue updatedTransactions:(nonnull NSArray<SKPaymentTransaction *> *)transactions {
    
    for (SKPaymentTransaction *transaction in transactions) {
        switch (transaction.transactionState) {
            case SKPaymentTransactionStatePurchasing:
                NSLog(@"Purchasing");
                break;
                
            case SKPaymentTransactionStatePurchased:
                {
                    long long milliseconds = (long long)(transaction.transactionDate.timeIntervalSince1970 * 1000.0);
                    NSString *purchaseTime = [NSString stringWithFormat:@"%lld", milliseconds];
                    NSLog(@"%lld", milliseconds);
                    NSLog(@"purchaseTime : %@", purchaseTime);
                    NSLog(@"transactionIdentifier : %@", transaction.transactionIdentifier);
                    NSLog(@"productIdentifier : %@", transaction.payment.productIdentifier);
                    NSLog(@"quantity : %@", [@(transaction.payment.quantity) stringValue]);
                    NSLog(@"getReceiptCode : %@", [self getReceiptCode]);
                    NSLog(@"Purchase completed");                    
                    
                    [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                }
                break;
                
            case SKPaymentTransactionStateRestored:
                NSLog(@"Restored");
                [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                break;
                
            case SKPaymentTransactionStateFailed:
                NSLog(@"Purchase failed %@", transaction.error);
                //[[SKPaymentQueue defaultQueue] finishTransaction:transaction];
                break;
                
            case SKPaymentTransactionStateDeferred:
            {
                break;
            }
                
            default:
                break;
        }
    }
}

- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
{
    NSUInteger count = [response.products count];
    
    if (count > 0) {
        _productList = response.products;
        
        for(SKProduct *skProduct in _productList)
        {
            NSLog(@"Found product: %@ – Product: %@ – Price: %0.2f", skProduct.productIdentifier, skProduct.localizedTitle, skProduct.price.floatValue);
        }
    } else {
        NSLog(@"No product or complete contract and tax financial transaction items.");
    }
}

@end
```



## 영수증 검증

* 서버가 없다면 클라이언트에서 구현해도 된다.
* 클라나 서버 둘중 한군데서 처리하면 되고 방식은 동일하다.
* 결제처리가 완료된 후 호출
* 검증url은 샌드박스용 앱스토어용 두가지가 있다.



```objective-c
-(void)verifyReceipt
{
    NSURL *receiptURL = [[NSBundle mainBundle] appStoreReceiptURL];
    NSData *receipt = [NSData dataWithContentsOfURL:receiptURL];
    if (!receipt) {
        NSLog(@"no receipt info");
    }
    
    // Create the JSON object that describes the request
    NSError *error;
    NSDictionary *requestContents = @{
        @"receipt-data": [receipt base64EncodedStringWithOptions:0]
    };
    NSData *requestData = [NSJSONSerialization dataWithJSONObject:requestContents
                                                          options:0
                                                            error:&error];
    
    if (!requestData) {
        NSLog(@"%@", error);
    }
    
    // Create a POST request with the receipt data.
    // https://sandbox.itunes.apple.com/verifyReceipt , https://buy.itunes.apple.com/verifyReceipt
    NSURL *storeURL = [NSURL URLWithString:@"https://sandbox.itunes.apple.com/verifyReceipt"];
    NSMutableURLRequest *storeRequest = [NSMutableURLRequest requestWithURL:storeURL];
    [storeRequest setHTTPMethod:@"POST"];
    [storeRequest setHTTPBody:requestData];
    
    [[NSURLSession sharedSession] dataTaskWithRequest:storeRequest
                                     completionHandler:^(NSData * _Nullable data,
                                                         NSURLResponse * _Nullable response,
                                                         NSError * _Nullable error)
    {
        if(error)
        {
            NSLog(@"%@", error);
        }else{
            NSError *error;
            NSDictionary *respDict = [NSJSONSerialization JSONObjectWithData:data options:0 error:&error];
            if (!respDict) {
                NSLog(@"%@", error);
            }
            NSLog(@"%@", respDict);
        }
    }];
}
```

