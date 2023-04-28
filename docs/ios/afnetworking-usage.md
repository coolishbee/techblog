

## AFNetworking 설치

```
pod 'AFNetworking', '~> 3.0'
```

## Example

### GET

```
- (void) getCarriersTracks:(NSString *)carrier_id
                 onSuccess:(void (^)(NSString * _Nullable))onSuccess
                 onFailure:(void (^)(NSError * _Nonnull))onFailure
{
    long long trackId = 1111111111111;
    NSString *strUrl = [NSString stringWithFormat:@"%@/carriers/%@/tracks/%lld", baseURL, @"kr.epost", trackId];
    NSURL *URL = [NSURL URLWithString:strUrl];
    
    //NSString *strLog = [Log objectToJsonString:[info toDictionary]];
    [Log print:@"GET" url:strUrl];
    
    AFHTTPSessionManager *manager = [[AFHTTPSessionManager alloc]initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    manager.requestSerializer = [AFJSONRequestSerializer serializer];
    [manager.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    
    [manager    GET:URL.absoluteString
         parameters:nil
           progress:nil
            success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable resp)
    {
        
        //NSLog(@"Carriers Tracks GET JSON: %@", resp);
        NSDictionary *respDict = resp;
        NSString *strLog = [Log objectToJsonString:respDict];
        [Log print:@"200" url:strUrl data:strLog];
        
        onSuccess(strLog);
    }failure:^(NSURLSessionDataTask *oper, NSError *err)
    {
        [Log print:[@(err.code) stringValue] url:strUrl data:err];
        
        NSLog(@"Error: %@", err);
        onFailure(err);
    }];
}
```

### POST

```
- (void) login:(NSString *)type
     onSuccess:(void (^)(RespLogin * _Nonnull))onSuccess
     onFailure:(void (^)(NSError * _Nonnull))onFailure
{
    NSString *strUrl = [NSString stringWithFormat:@"%@/auth", localURL];
    NSURL *reqURL = [NSURL URLWithString:strUrl];
    
    ReqLogin* loginBody = [[ReqLogin alloc] init];
    loginBody.login_type = type;
    loginBody.login_token = @"id_token";
        
    NSString *strLog = [Log objectToJsonString:[loginBody toDictionary]];
    [Log print:@"POST" url:strUrl data:strLog];    
    
    AFHTTPSessionManager *manager = [[AFHTTPSessionManager alloc]initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    manager.requestSerializer = [AFJSONRequestSerializer serializer];
    [manager.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    
    [manager    POST:reqURL.absoluteString
          parameters:[loginBody toDictionary]
            progress:nil
             success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable resp)
    {
        NSDictionary *respDict = resp;
        NSError *error;
        NSString *strLog = [Log objectToJsonString:respDict];
        
        RespLogin* respLogin = [[RespLogin alloc] initWithDictionary:respDict error:&error];
        NSString *resultCode = [NSString stringWithFormat:@"%ld", (long)respLogin.code];
        [Log print:resultCode url:strUrl data:strLog];        
        NSLog(@"Error log: %@", error);
        
        onSuccess(respLogin);
    }failure:^(NSURLSessionDataTask *oper, NSError *error)
    {
        NSLog(@"Error log: %@", error);
        onFailure(error);
    }];
    
}
```

!!! Github
    [https://github.com/coolishbee/AFNetworkingExample](https://github.com/coolishbee/AFNetworkingExample)