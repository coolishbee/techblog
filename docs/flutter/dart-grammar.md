
## Async

```dart
const oneSecond = Duration(seconds: 1);
// ···
Future<void> printWithDelay(String message) async {
  await Future.delayed(oneSecond);
  print(message);
}
```

!!! 공식사이트
    [https://dart.dev/language](https://dart.dev/language)