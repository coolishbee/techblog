---
comments: true
---

# ARC

메모리 관리를 참조카운팅 방식으로 하는 기술로써 Swift는 자동 참조 계산(ARC)을 사용하여 앱의 메모리 사용량을 추적하고 관리합니다. 
대부분의 경우 이는 Swift에서 메모리 관리가 "그냥 작동"한다는 의미이며 메모리 관리에 대해 직접 생각할 필요가 없습니다. 
ARC는 해당 인스턴스가 더 이상 필요하지 않을 때 클래스 인스턴스에서 사용하는 메모리를 자동으로 해제합니다.

## 강한 참조

ARC에 의해 관리되며, 참조 횟수가 증가하고 이에 대한 참조가 개체 수명 동안 유지된다는 것을 의미합니다.
카운트가 0에 도달하면 메모리가 해제될 수 있습니다.
문제는 객체간의 서로 참조할 경우인데 이런 경우엔 두 개체를 모두 해제할 수 없으며 메모리 누수가 발생합니다.

## weak

객체를 가리키고 있지만 참조 횟수를 늘리지 않는다는 의미입니다.
부모-자식 관계를 만들 때 자주 사용됩니다. 부모는 자식에 대해 강한 참조를 갖고 있지만 자식은 부모에 대해 약한 참조만을 갖고 있습니다.

## unowned

weak 와 유사하지만 optional 변수는 허용되지 않는다. 반대로 weak는 optional 이어야 한다.

## 순환 참조

https://velog.io/@wonbi92/Retain-Cycle

순환참조 확인할 방법

https://medium.com/@LeeZion94/strong-reference-cycle-8a88bdd8424b

https://jooeungen.tistory.com/entry/iOSSwift-Reference-Count-체크-RetainCycle-확인하기