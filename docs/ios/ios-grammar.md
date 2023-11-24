

## 배열 인덱스와 값 출력
```swift
let seoul = ["Jane", "Kim"]
for (idx, name) in seoul.enumerated() {
    print("\(idx), \(name)")
}
```

## 배열 일치하는 값의 인덱스 찾기

```swift
let seoul = ["Jane", "Kim"]
print("\(seoul.firstIndex(of: "Kim")!)")
```

## 배열 내림차순으로 정렬후 인덱스 순으로 배열 반환

```swift
let emergency = [3, 76, 24]
let result = emergency.map {
    emergency.sorted(by: > ).firstIndex(of: $0)!
}
print(result)
```

## 배열 내림차순으로 정렬후 순차적으로 값 뽑기

```swift
let emergency = [3, 76, 24]
for i in emergency.sorted(by: >) {
    print(i)
}
```

## 배열 중복 인덱스 제거

```swift
userInfo.removeAll(where: { $0.name == "민이" })
```

## 배열의 특정 index 찾기

```swift
if let index = userInfo.firstIndex(where: { $0.name == "민이" }) {
    print(userInfo[index].phone)
}
```

## unix 와 같은 13자리 timestamp 얻기

=== "swift"

    ```swift
    // ex1)
    let currentDate = NSNumber(value:Int64(Date().timeIntervalSince1970 * 1000))

    // ex2)
    extension Date {
        
        func toMillis() -> NSNumber {
            return NSNumber(value:Int64(timeIntervalSince1970 * 1000))
        }
        
        var timestamp: Int64 {
            Int64(timeIntervalSince1970 * 1000)
        }
    }
    ```
=== "objc"

    ```objc
    long long milliseconds = (long long)([[NSDate date] timeIntervalSince1970] * 1000.0);
    ```

## CodingKeys 사용법

=== "swift"

    ```swift
    struct Login: Codable {
        let code: UInt
        let msg: String
        let data: LoginInfo
        enum CodingKeys: String, CodingKey {
            case code = "code"
            case msg = "msg"
            case data = "data"
        }
    }

    struct LoginInfo: Codable {
        let userID: String
        let username: String
        enum CodingKeys: String, CodingKey {
            case userID = "userID"
            case username = "username"
        }
    }
    ```

## string to enum

=== "swift"
```swift
public enum LoginType: String {
    case google="GOOGLE"
    case facebook="FACEBOOK"
    case apple="APPLE"
    case guest="GUEST"
}

print(LoginType.google.rawValue) //GOOGLE
```

!!! 출처
    [https://stackoverflow.com/questions/24701075/swift-convert-enum-value-to-string](https://stackoverflow.com/questions/24701075/swift-convert-enum-value-to-string)

## int to enum

=== "swift"
```swift
public enum LanguageCode : Int{
    case ko
    case en
    case ja
    case zhcn
    case zhtw
    case th
    case vi
    case es
    case pt
    case fr
    case de
    case ru
    
    var description: String {
        return String(describing: self)
    }
}

print(languageCode.rawValue) //0
print(languageCode.description) //ko
```

!!! 출처
    [https://stackoverflow.com/questions/28461133/swift-enum-both-a-string-and-an-int](https://stackoverflow.com/questions/28461133/swift-enum-both-a-string-and-an-int)

## UUID 생성하기

=== "swift"

    ```swift
    UUID.init().uuidString
    ```
=== "objc"

    ```objc
    + (NSString *) createUUID
    {
        NSString *uuid = [[NSUUID UUID] UUIDString];

        return uuid;
    }
    //output : 61237D34-C6ED-4AA7-8151-A42053C953B4
    ```

## [objc] NSURL 사용법

### 1. NSURL 속성

-   URL: [http://www.testUrl.com/notice/contents?a=1&b=2&c=3](http://www.testUrl.com/notice/contents?a=1&b=2&c=3) (설명을 위한 임시 URL)
-   scheme: http (:// 이전의 텍스트)
-   host: [www.testUrl.com](http://www.testUrl.com) (://과 맨 처음 / 사이의 텍스트)
-   path: /notice/contents (맨 처음 /와 ? 사이의 텍스트)
-   query: a=1&b=2&c=3 (? 이후의 텍스트)

### 예제

```
NSString *strUrl = @"https://www.api.com/api/info?t=123&sign=test123";
NSURL *reqURL = [NSURL URLWithString:strUrl];
NSLog(@"%@", [reqURL host]);
NSLog(@"%@", [reqURL path]);
NSLog(@"%@", [reqURL query]);
```

### 2. NSURL 에서 NSString 으로 변환

```
NSString *myString = myURL.absoluteString;
```

!!! 출처
    [https://stackoverflow.com/questions/8082719/convert-an-nsurl-to-an-nsstring](https://stackoverflow.com/questions/8082719/convert-an-nsurl-to-an-nsstring)

## md5 encode

=== "swift"

    ```swift
    import CommonCrypto

    func MD5(_ input: String) -> String? {
        let length = Int(CC_MD5_DIGEST_LENGTH)
        var digest = [UInt8](repeating: 0, count: length)
        if let d = input.data(using: .utf8) {
            _ = d.withUnsafeBytes { body -> String in
                CC_MD5(body.baseAddress, CC_LONG(d.count), &digest)
                return ""
            }
        }
        return (0..<length).reduce("") {
            $0 + String(format: "%02x", digest[$1])
        }
    }
    ```

=== "objc"

    ```objc
    #import <CommonCrypto/CommonDigest.h>

    + (NSString *) md5:(NSString *) input
    {
        const char *cStr = [input UTF8String];
        unsigned char digest[CC_MD5_DIGEST_LENGTH];
        CC_MD5( cStr, (CC_LONG)strlen(cStr), digest ); // This is the md5 call

        NSMutableString *output = [NSMutableString stringWithCapacity:CC_MD5_DIGEST_LENGTH * 2];

        for(int i = 0; i < CC_MD5_DIGEST_LENGTH; i++)
        [output appendFormat:@"%02x", digest[i]];

        return  output;
    }
    ```

## [objc] 콜백 리스너 예제(Blocks Sample Code)

objc 에서 콜백함수에 대한 이벤트리스너를 만드려면

1. block(^) 
2. selector 
3. noti 
4. delegate 

가 있다고 한다. 현재 상황에서 나에게 필요한건 block 이었기 때문에 혹시 필요하신 분들을 위해 샘플코드를 제공합니다.

=== "sample"

    ```
    @implementation
    -(void)doSomething:(void (^)(BOOL, int))completionBlock
    {
        NSLog(@"Do Something first");

        completionBlock(YES, 1);
        completionBlock(NO, 2);

        NSLog(@"Then may be something else");
    }

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        [self doSomething:^(BOOL isOk, int code) {
            //code
        }]        
    }
    @end
    ```
=== "typedef"
    ```
    헤더
    typedef void (^responseBlock)(BOOL, int);

    @implementation
    -(void)doSomething:(responseBlock)completionBlock
    {
        NSLog(@"Do Something first");

        completionBlock(YES, 1);
        completionBlock(NO, 2);

        NSLog(@"Then may be something else");
    }

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        [self doSomething:^(BOOL isOk, int code) {
            //code
        }]        
    }
    @end
    ```

## isKindOfClass 사용법

딕셔너리에 string int long 등이 있을때 element 타입체크하기.
=== "objc"
```
for(NSString *key in mDict)
{
  id value = [mDict objectForKey:key];
  if([value isKindOfClass:[NSString class]]) {
    NSString *temp = (NSString*)value;
    NSLog(@"%@",temp);
  }
  else if([value isKindOfClass:[NSNumber class]]) {
    NSNumber *numTemp = (NSNumber*)value;
    NSString *strTemp = [numTemp stringValue];
    NSLog(@"%@",strTemp);
  }
}
```