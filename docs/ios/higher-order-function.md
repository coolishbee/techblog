# 고차함수

## map

## filter

## compactMap

## flatMap

## zip map reduce

배열 각 연소들의 합
```swift
let absolutes = [1, 2, 3, 4, 5]
let signs = [true, false, true]
result = zip(absolutes, signs)
    .map { $1 ? $0 : -$0 }
    .reduce(0, +)
print(result)
```

## reduce

배열 각 연소들의 합
```swift
let absolutes = [1, 2, 3, 4, 5]
let signs = [true, false, true]
result = (0..<absolutes.count).map {    
    signs[$0] ? absolutes[$0] : -absolutes[$0]
}.reduce(0, +)
print(result)
```

정수 n의 약수들의 합
```swift
let result = Array(1...n).filter{
    n % $0 == 0
}.reduce(0, +)
```